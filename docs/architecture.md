# AURO — System Architecture

> **Document Type:** Architecture Reference  
> **Version:** 1.0  
> **Status:** Draft — to be updated as implementation progresses

---

## Overview

AURO is built on a **five-layer modular architecture** where each layer has clearly defined responsibilities and communicates with adjacent layers through well-defined interfaces.

```
┌─────────────────────────────────────────────────────┐
│              LAYER 1: PRESENTATION                   │
│         React + TypeScript + Tailwind CSS            │
│  Dashboards · Resource Mgmt · Timetable · Approval  │
└───────────────────────┬─────────────────────────────┘
                        │ HTTP / WebSocket
┌───────────────────────▼─────────────────────────────┐
│              LAYER 2: APPLICATION                    │
│                   FastAPI (Python)                   │
│  Auth · RBAC · REST APIs · WebSocket · Audit        │
└──────────────┬─────────────────┬────────────────────┘
               │ Function calls  │ DB sessions
┌──────────────▼──────────┐  ┌───▼──────────────────┐
│   LAYER 3: PROCESSING   │  │   LAYER 4: DATA       │
│   Python DSA + OS MP    │  │      PostgreSQL        │
│  Graph · BFS · CSP      │  │  Tables · Indexes     │
│  Workers · IPC · Queues │  │  Transactions · ACID  │
└─────────────────────────┘  └──────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│              LAYER 5: COMMUNICATION                  │
│       REST · WebSocket · Multiprocessing IPC        │
└─────────────────────────────────────────────────────┘
```

---

## Layer Descriptions

### Layer 1 — Presentation

**Technology:** React 18 + TypeScript + Vite + Tailwind CSS

| Component | Responsibility |
|---|---|
| Admin Dashboard | System-wide stats, jobs, conflicts |
| Resource Management | CRUD for all university entities |
| Timetable View | Weekly/daily/role-filtered schedule display |
| Conflict Center | Conflict visualization and recovery options |
| Simulation Center | What-if analysis and alternative comparison |
| Approval Center | Version-validated administrator approval UI |
| Faculty / Student Portals | Role-scoped read-only timetable views |

**State Management:**
- Local state: `useState`, `useReducer`
- Server state: TanStack Query (caching, refetching, mutations)
- Real-time: WebSocket context provider

---

### Layer 2 — Application

**Technology:** Python 3.11+ · FastAPI · Pydantic v2 · SQLAlchemy 2.x

| Module | Responsibility |
|---|---|
| `api/routes/auth.py` | JWT login, registration, /me |
| `api/routes/departments.py` | Department CRUD |
| `api/routes/faculty.py` | Faculty management |
| `api/routes/schedules.py` | Generation trigger, retrieval |
| `api/routes/simulations.py` | Simulation lifecycle |
| `api/routes/approvals.py` | Versioned transactional commit |
| `core/security.py` | JWT, password hashing, RBAC |
| `services/schedule_service.py` | Orchestrates DSA engine calls |
| `services/approval_service.py` | Transactional approval logic |
| `websocket/manager.py` | Connection registry + broadcast |

---

### Layer 3 — Processing

**Technology:** Python `multiprocessing` module

```
Coordinator Process
       │
       ├── Job Queue (Priority Queue)
       │         │
       │    ┌────▼────────────────────────────┐
       │    │      Worker Pool (N workers)     │
       │    │  Worker-1  Worker-2  Worker-N   │
       │    └────┬────────────────────────────┘
       │         │
       └── Result Queue
                 │
           Result Handler
```

**DSA modules called by workers:**
- `dsa/graph.py` — Conflict graph (faculty, batch, room edges)
- `dsa/bfs_impact.py` — Multi-hop ripple impact analysis around disruptions
- `dsa/priority_queue.py` — Min-heap priority dispatching for recovery tasks
- `dsa/hash_maps.py` — O(1) in-memory lookups for activities, slots, and resources
- `dsa/csp.py` — Localized CSP subproblem modeling
- `dsa/backtracking.py` — Recursive candidate search with Forward Checking pruning
- `dsa/optimizer.py` — Minimum-disruption soft constraint scoring (displacement penalty)
- `dsa/incremental_scheduler.py` — Fast localized recovery engine

---

### Layer 4 — Data

**Technology:** PostgreSQL 15+

Key design decisions:
- Full ACID compliance for schedule commits
- Optimistic locking via `schedule_version_id`
- GiST exclusion constraints for time-overlap prevention
- JSONB for flexible metadata (`previous_values`, `new_values` in AuditLog)
- Schedule versioning — history is never overwritten
- Transactional outbox for notifications

---

### Layer 5 — Communication

| Channel | Protocol | Purpose |
|---|---|---|
| Client ↔ FastAPI | HTTPS / REST | Resource CRUD, schedule retrieval |
| Client ↔ FastAPI | WebSocket | Real-time job progress, notifications |
| Coordinator ↔ Workers | `multiprocessing.Queue` | Job dispatch and result collection |
| Workers ↔ Database | SQLAlchemy session | Persist scheduling results |
| Notification Worker ↔ DB | SQLAlchemy session | Outbox polling and delivery |

---

## Process Communication Diagram

```
                    ┌──────────────────────┐
                    │   FastAPI Server      │
                    │  (Application Layer)  │
                    └──────────┬───────────┘
                               │  Enqueue job via IPC
                    ┌──────────▼───────────┐
                    │    Coordinator        │
                    │  (OS Engine Process)  │
                    └──────────┬───────────┘
                               │  multiprocessing.Queue
              ┌────────────────▼─────────────────────┐
              │            Job Queue (Priority)        │
              └────────┬──────────────┬───────────────┘
                       │              │
              ┌────────▼──────┐  ┌────▼──────────┐
              │   Worker 1    │  │   Worker 2     │
              │  DSA Engine   │  │  DSA Engine    │
              └────────┬──────┘  └────┬──────────┘
                       │              │
              ┌────────▼──────────────▼────────────┐
              │          Result Queue               │
              └────────────────┬───────────────────┘
                               │
                    ┌──────────▼───────────┐
                    │   Result Handler      │
                    │  → Update job status  │
                    │  → Store simulation   │
                    │  → Broadcast WS event │
                    └──────────────────────┘
```

---

## Scheduling Workflow

```
1. Admin triggers generation
         │
         ▼
2. FastAPI creates Job record (PENDING)
         │
         ▼
3. Coordinator receives job → enqueues (QUEUED)
         │
         ▼
4. Worker picks up job (RUNNING)
         │
         ▼
5. Ingest Pre-built Schedule & Build Conflict Graph
         │
         ▼
6. BFS Impact Analysis (radius k) on Disrupted Resource
         │
         ▼
7. Localized CSP: Reassign impacted activities
   │   ├── Min-Heap Priority Queue: Order tasks by urgency
   │   ├── Forward Checking: Prune invalid domains
   │   └── Backtracking: Search minimal-disruption alternatives
         │
         ▼
8. Validate hard constraints
         │
         ▼
9. Score soft constraints → rank alternatives
         │
         ▼
10. Persist ScheduleVersion + Assignments (COMPLETED)
         │
         ▼
11. Broadcast WebSocket event to admin
```

---

## Transaction and Recovery Workflow

```
1. Admin approves simulation alternative
         │
         ▼
2. Backend: verify user role (ADMIN)
         │
         ▼
3. Lock current ScheduleVersion row
         │
         ▼
4. Validate: simulation.base_version == current active version
   └── FAIL → reject with 409 Conflict
         │
         ▼
5. BEGIN TRANSACTION
         │
         ├── Validate all hard constraints for proposed changes
         ├── Insert new ScheduleVersion (ACTIVE)
         ├── Insert/Update Assignments
         ├── Deactivate previous version
         ├── Insert AuditLog records
         └── Insert Notification outbox records
         │
         ▼
6. COMMIT
         │
         ▼
7. Notification Worker polls outbox → delivers notifications
         │
         ▼
8. WebSocket broadcast: new active version
```

---

## Security Architecture

```
Client Request
     │
     ▼
┌──────────────────────────────────────┐
│  FastAPI Middleware Stack            │
│  ┌────────────────────────────────┐ │
│  │ CORS Middleware                │ │
│  ├────────────────────────────────┤ │
│  │ Rate Limiting (sensitive APIs) │ │
│  ├────────────────────────────────┤ │
│  │ JWT Authentication Dependency  │ │
│  ├────────────────────────────────┤ │
│  │ RBAC Permission Check          │ │
│  ├────────────────────────────────┤ │
│  │ Pydantic Input Validation      │ │
│  └────────────────────────────────┘ │
└──────────────────────────────────────┘
     │
     ▼
Route Handler → SQLAlchemy ORM (parameterized) → PostgreSQL
```

---

## Deployment Architecture

```
docker-compose.yml
      │
      ├── postgres:15
      │       Port: 5432
      │       Volume: postgres_data
      │
      ├── backend (FastAPI)
      │       Port: 8000
      │       Depends: postgres
      │       Workers: uvicorn + multiprocessing pool
      │
      └── frontend (Vite → Nginx)
              Port: 3000
              Proxy: /api/* → backend:8000
```

---

*This document will be updated as each phase is implemented and verified.*

