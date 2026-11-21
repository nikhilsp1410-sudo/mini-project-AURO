# AURO — Additional Details

> **Document Type:** Supplementary Specification  
> **Status:** ✅ RESOLVED — All details provided in `COMPLETE_SPECIFICATION.md`  
> **Last Updated:** 2026-11-06

---

## Status

All supplementary details have been provided by the project owner as part of the **complete finalized specification**.

See [`COMPLETE_SPECIFICATION.md`](./COMPLETE_SPECIFICATION.md) — this is the **single source of truth** for all project development decisions.

---

## Summary of What Was Clarified

The complete specification (v2.0) confirms and finalizes the following, superseding any "pending" items from this document:

### ✅ Technology Stack (Finalized)

| Concern | Decision |
|---|---|
| Frontend | React + TypeScript + Tailwind CSS + Vite |
| Backend | Python + FastAPI + Pydantic |
| Database | PostgreSQL only (no MongoDB, no SQLite) |
| Auth | JWT + Argon2/bcrypt + RBAC |
| IPC | Python `multiprocessing.Queue` |
| Deployment | Docker + Docker Compose |
| Real-time | WebSockets (FastAPI) |

### ✅ DSA Algorithms (Finalized — Do Not Replace)

| Algorithm | Purpose |
|---|---|
| Graph Theory (Conflict Graph) | Model incompatible scheduling activities |
| BFS (Breadth-First Search) | Multi-hop ripple impact analysis for disrupted resources |
| CSP (Constraint Satisfaction Problem) | Model valid assignment domains and rules |
| Backtracking | Search for valid assignments |
| Forward Checking | Prune domains early to avoid dead-ends |
| Priority Queue | Activity and job selection ordering |
| Hash Maps | O(1) resource lookups throughout scheduling |

### ✅ Scheduling Logic (Finalized)

- Two-stage scheduling: **Feasibility first, then Soft-Constraint Optimization**
- Incremental rescheduling: **Minimize changes, preserve unaffected assignments**
- Objective function: $C = w_1 D + w_2 P + w_3 G + w_4 U$ (configurable weights)
- Hard constraints **never** violated for soft-constraint improvement
- Time overlap checked on **actual clock intervals**, not slot IDs

### ✅ OS Process Architecture (Finalized)

- Actual Python `multiprocessing` (not threading)
- Logical roles: Coordinator, Conflict Detector, Scheduling Worker, Result Handler, Notification Worker
- Job priority levels: Emergency (0) → High (1) → Medium (2) → Normal (3) → Low (4)
- Aging to prevent starvation
- Mutex + Semaphores for shared resource protection
- Deadlock prevention via consistent lock ordering + timeouts
- Worker failure detection + bounded retry with idempotency

### ✅ Database Design (Finalized)

- 19 tables fully defined (see [`database_schema.md`](./database_schema.md))
- **NotificationOutbox** table added (transactional outbox pattern)
- GiST exclusion constraints for time-overlap prevention
- Optimistic locking for schedule approval
- `SELECT FOR UPDATE` during commit transaction
- Partial unique index: only one `status = 'active'` ScheduleVersion at a time

### ✅ Security (Finalized)

- Bcrypt or Argon2 — no plaintext passwords
- JWT for all endpoints
- RBAC enforced server-side (not just UI hiding)
- WebSocket auth via JWT query parameter
- Rate limiting on sensitive endpoints
- Pydantic for all input validation

### ✅ Sample Data (Finalized)

- 2 departments, 10 faculty, 5 batches, 10 courses, 5 classrooms, 3 labs
- 4 demo accounts (admin, coordinator, faculty, student)
- Lab 3 unavailability scenario fully demonstrable from seed data

### ✅ Development Phases (Finalized — 12 Phases)

| Phase | Focus |
|---|---|
| 1 | Requirements and architecture |
| 2 | Database schema and migrations |
| 3 | Resource management |
| 4 | DSA algorithm implementation |
| 5 | Initial timetable generation |
| 6 | OS multiprocessing and workers |
| 7 | Conflict detection and recovery |
| 8 | Simulation and alternatives |
| 9 | Approval, versioning and transactions |
| 10 | Frontend dashboards |
| 11 | WebSockets and notifications |
| 12 | Testing and deployment |

---

## Items Still Pending

The following items were not specified in the complete specification and may be clarified during implementation:

| Item | Default Assumption | Needs Confirmation? |
|---|---|---|
| University working hours | Mon–Sat, 9:00–17:00 | Optional |
| Session duration options | 1 hour standard, 2 hour lab | Optional |
| Max sessions per day per batch | 5–6 | Optional |
| Branding / color scheme | Professional blue-grey academic | Optional |
| Cloud deployment target | Local Docker only for now | Optional |
| Email delivery for notifications | In-app only initially | Optional |
| PDF timetable export | Not in current scope | Optional |
| Multi-language support | English only | Optional |

These defaults will be used unless the project owner specifies otherwise.

---

*All major design decisions are now finalized. Refer to `COMPLETE_SPECIFICATION.md` for the authoritative project definition.*
