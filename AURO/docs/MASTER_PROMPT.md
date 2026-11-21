# AURO — Autonomous University Resource Orchestrator
## Master Implementation Specification

> **Document Type:** Master Implementation Prompt  
> **Version:** 1.0  
> **Status:** Active  
> **Last Updated:** 2026-11-06

---

## Table of Contents

1. [Role and Objective](#1-role-and-objective)
2. [Project Purpose](#2-project-purpose)
3. [Technology Stack](#3-technology-stack)
4. [System Architecture](#4-system-architecture)
5. [Engine 1 — DSA Engine (35%)](#5-engine-1--dsa-engine-35)
6. [Engine 2 — DBMS Engine (25%)](#6-engine-2--dbms-engine-25)
7. [Engine 3 — OS Engine (20%)](#7-engine-3--os-engine-20)
8. [Engine 4 — Web & Networks (20%)](#8-engine-4--web--networks-20)
9. [End-to-End Scenario](#9-most-important-end-to-end-scenario)
10. [Functional Requirements](#10-functional-requirements)
11. [Non-Functional Requirements](#11-non-functional-requirements)
12. [Security Requirements](#12-security-requirements)
13. [Testing Requirements](#13-testing-requirements)
14. [UX & Design Requirements](#14-user-experience-and-design-requirements)
15. [Project Folder Structure](#15-project-folder-structure)
16. [Development Strategy](#16-development-strategy)
17. [Sample Data & Demonstration](#17-sample-data-and-demonstration)
18. [Documentation Requirements](#18-documentation-requirements)
19. [Strict Implementation Rules](#19-strict-implementation-rules)
20. [Final Acceptance Criteria](#20-final-acceptance-criteria)

---

## 1. Role and Objective

**Act as:** Senior full-stack software architect, Python backend engineer, database architect, DSA specialist, operating systems engineer, frontend developer and software testing engineer.

**Task:** Design, implement, test and deliver a complete, functional, production-structured university resource management and intelligent scheduling application called **AURO — Autonomous University Resource Orchestrator**.

This must be a **real, working full-stack application**, not merely a UI prototype or a collection of disconnected modules.

### Engineering Domain Contributions

| Domain | Contribution |
|---|---|
| Data Structures & Algorithms (DSA) | 35% |
| Database Management Systems (DBMS) | 25% |
| Operating Systems (OS) | 20% |
| Web Technologies & Computer Networks | 20% |

> These percentages represent the project's **academic and architectural contribution**, not runtime resource allocation.

### Ground Rules

- Do **not** replace the selected technologies or algorithms with simpler alternatives without explaining the technical reason and obtaining approval.
- Do **not** create fake functionality, placeholder buttons, simulated database operations or hardcoded scheduling results.
- Every major feature must be connected to its **actual backend implementation**.

---

## 2. Project Purpose

AURO is an **intelligent university resource management and timetable orchestration platform**.

### Managed Entities

- University departments
- Faculty members
- Student batches
- Courses and subjects
- Classrooms
- Laboratories
- Laboratory equipment
- Faculty availability
- Room availability
- Time slots
- Academic scheduling activities
- Timetable assignments
- Resource unavailability
- Schedule versions
- Simulation results
- Audit logs

### Core Differentiator

AURO must **not** behave like an ordinary timetable generator. It must support:

1. Initial timetable generation
2. Automatic conflict detection
3. Resource availability monitoring
4. Incremental schedule recovery
5. Alternative schedule generation
6. What-if simulation
7. Alternative comparison
8. Administrator approval
9. Transactional schedule updates
10. Notifications and audit tracking

**Central Principle:**
> Generate a valid timetable, detect disruptions, recover intelligently and minimize changes to unaffected assignments.

---

## 3. Technology Stack

### Frontend

| Technology | Purpose |
|---|---|
| React | UI framework |
| TypeScript | Type-safe JavaScript |
| Vite | Build tool |
| Tailwind CSS | Utility-first styling |
| React Router | Client-side routing |
| TanStack Query | Server-state management |
| Recharts / Chart.js | Dashboard analytics |
| Lucide React | Icon library |

### Backend

| Technology | Purpose |
|---|---|
| Python | Primary language |
| FastAPI | Web framework |
| Pydantic | Data validation |
| SQLAlchemy 2.x | ORM |
| Alembic | Database migrations |
| asyncpg / psycopg2 | PostgreSQL driver |
| Python Multiprocessing | Process management |

### Database

| Technology | Purpose |
|---|---|
| PostgreSQL | Primary database |
| JSONB | Flexible metadata fields only |

> Do **not** use MongoDB as the primary database.  
> Do **not** use SQLite as a replacement for PostgreSQL.

### Process Management

- Python `multiprocessing`
- Multiprocessing `Queue` for IPC
- Worker Pool
- Mutex and Semaphores
- Priority-based job scheduling with Aging
- Worker health monitoring
- Bounded retry mechanisms

### Authentication

- JWT tokens
- Argon2 or bcrypt password hashing
- Role-Based Access Control (RBAC)

### Deployment

- Docker
- Docker Compose
- Environment variables via `.env`
- Git-compatible project structure

---

## 4. System Architecture

### Layer 1: Presentation Layer

**Technology:** React + TypeScript + Tailwind CSS

Responsible for:
- Admin dashboard
- Department coordinator dashboard
- Faculty timetable interface
- Student timetable interface
- Resource management interface
- Scheduling interface
- Conflict management
- Simulation interface
- Approval interface

### Layer 2: Application Layer

**Technology:** FastAPI

Responsible for:
- Authentication & Authorization
- Resource management
- Timetable requests
- Schedule generation requests
- Simulation management
- Approval and commit operations
- Notifications
- Audit logging

### Layer 3: Processing Layer

**Technology:** Python DSA modules + OS Multiprocessing

Responsible for:
- Graph construction
- DSATUR algorithm
- CSP modeling
- Backtracking with Forward Checking
- Alternative generation
- Conflict detection
- Scheduling workers
- Job management

### Layer 4: Data Layer

**Technology:** PostgreSQL

Responsible for:
- Resource records
- Timetable assignments
- Schedule versions
- Simulation records
- Audit logs
- Notifications
- Persistent application data

### Layer 5: Communication Layer

Responsible for:
- REST APIs (FastAPI routes)
- WebSocket connections
- Inter-process communication
- Worker status updates
- Real-time progress updates

---

## 5. Engine 1 — DSA Engine (35%)

### 5.1 Conflict Graph

Represent scheduling activities as **vertices** and conflicts as **edges**.

**Edge conditions (two activities share):**
- Same faculty member
- Same student batch
- Same classroom
- Same laboratory
- Other mutually exclusive constraints

**Required graph operations:**
- Add / Remove vertex
- Add / Remove edge
- Retrieve neighbors
- Calculate degree
- Check adjacency
- Generate conflict information

> The conflict graph must be constructed from **actual scheduling data**. Do not hardcode relationships.

### 5.2 DSATUR Algorithm

Implement the **Degree of Saturation** algorithm for prioritizing constrained activities.

**Selection criteria:**
1. Saturation degree (primary)
2. Vertex degree (tie-breaker)
3. Deterministic tie-breaking for reproducibility

DSATUR supports graph-colouring and slot-prioritization logic.

### 5.3 Constraint Satisfaction Problem

**Variables:** Scheduling activities / class sessions  
**Domains:** Feasible combinations of time slots + compatible rooms

#### Hard Constraints *(must never be violated)*

- No faculty member can teach two classes simultaneously
- No room can host two classes simultaneously
- No student batch can attend two classes simultaneously
- Room capacity must be sufficient
- Required equipment must be available
- Unavailable faculty or rooms cannot be assigned

#### Soft Constraints *(should be optimized)*

- Faculty preferred time slots
- Preferred rooms
- Reduced gaps between classes
- Balanced daily teaching loads
- Reduced timetable changes
- Better resource utilization

> **Rule:** Never trade away a hard constraint to improve a soft-constraint score.

### 5.4 Backtracking with Forward Checking

After assigning a variable:
1. Update remaining variables' domains
2. Remove impossible assignments
3. Check for empty domains → backtrack if detected
4. Restore domain changes correctly when backtracking

Features:
- Domain reduction
- Early dead-end detection
- Constraint validation
- Search limits and cancellation support

### 5.5 Priority Queue

Used for:
- Selecting highly constrained activities
- Managing urgent recovery jobs
- Processing activities with limited feasible choices

> Do not use a priority queue merely to claim it is implemented.

### 5.6 Hash Maps

Used for efficient lookup of:
- Faculty ID → faculty information
- Room ID → room information
- Batch ID → batch information
- Activity ID → scheduling state
- Resource ID → availability
- Assignment ID → timetable assignment

### 5.7 Scheduling Objective

**Stage A: Feasibility** — Satisfy every hard constraint.

**Stage B: Optimization** — Rank using weighted soft-constraint objective.

Minimize:
- Number of changed assignments
- Faculty preference violations
- Unnecessary idle periods
- Poor room utilization
- Unbalanced workloads

> Objective weights must be **configurable**. Hard constraints validated independently.

### 5.8 Incremental Rescheduling

When a resource becomes unavailable:
1. Identify assignments using that resource
2. Identify directly affected activities
3. Identify activities that may conflict with alternatives
4. Build the affected scheduling subproblem
5. Preserve unaffected assignments as fixed
6. Generate feasible alternatives
7. Minimize modified assignments
8. Validate the complete resulting timetable

> If incremental recovery is impossible, **report unresolved conflicts**. Do not silently return an invalid schedule.

### 5.9 Alternative Generation

Generate multiple valid candidate schedules. Each alternative must include:
- Changed / unchanged assignments
- New room and time-slot allocations
- Unresolved conflicts (if any)
- Soft-constraint score
- Number of modified activities
- Resource utilization information

Rank alternatives using configurable optimization criteria.

---

## 6. Engine 2 — DBMS Engine (25%)

### 6.1 Database Normalization

Design relational tables with appropriate normalization, avoiding unnecessary duplication. Use primary keys, foreign keys and meaningful relationships.

### 6.2 Database Schema

#### Department
```
department_id | department_name | department_code | created_at
```

#### User
```
user_id | full_name | email | password_hash | role | department_id | is_active | created_at
```

#### Faculty
```
faculty_id | user_id | department_id | designation | maximum_weekly_load
```

#### StudentBatch
```
batch_id | batch_name | department_id | semester | student_count | academic_year
```

#### Course
```
course_id | course_code | course_name | department_id | course_type | required_duration | requires_lab
```

#### Room
```
room_id | room_name | building | capacity | room_type | is_active
```

#### Laboratory
```
laboratory_id | room_id | laboratory_name | laboratory_type
```

#### Equipment
```
equipment_id | equipment_name | equipment_type | quantity
```

#### RoomEquipment
```
room_id | equipment_id | available_quantity
```

#### TimeSlot
```
slot_id | day_of_week | start_time | end_time
```

#### CourseOffering
```
offering_id | course_id | faculty_id | batch_id | sessions_per_week
```

#### FacultyAvailability
```
availability_id | faculty_id | slot_id | is_available
```

#### ResourceUnavailability
```
unavailability_id | resource_type | resource_id | start_datetime | end_datetime | reason | created_by
```

#### Assignment
```
assignment_id | schedule_version_id | offering_id | room_id | slot_id | session_date_or_day | status
```

#### ScheduleVersion
```
schedule_version_id | version_number | status | created_by | created_at | approved_at
```

#### Simulation
```
simulation_id | base_schedule_version_id | status | objective_score | created_by | created_at
```

#### SimulationAssignment
```
simulation_assignment_id | simulation_id | offering_id | proposed_room_id | proposed_slot_id | is_changed
```

#### AuditLog
```
audit_id | user_id | action | entity_type | entity_id | previous_values | new_values | timestamp
```

#### Notification
```
notification_id | user_id | title | message | notification_type | is_read | created_at
```

### 6.3 Indexing

**Index types:**
- B-tree indexes (default)
- Composite indexes for multi-column lookups
- Partial indexes for filtered queries

**Common index targets:**
- faculty_id, room_id, batch_id, course_id
- slot_id, schedule_version_id, simulation_id

> Use `EXPLAIN ANALYZE` to investigate important queries. Do not add indexes without considering query patterns and write overhead.

### 6.4 ACID Transactions

Schedule approval transaction flow:
1. Start transaction
2. Verify current schedule version
3. Validate proposed assignments
4. Apply approved changes
5. Create new schedule version
6. Create audit records
7. Create notification outbox records
8. **Commit**

> If any critical operation fails → **Roll back the entire transaction**.

### 6.5 Concurrency Control

- Optimistic locking
- Schedule-version validation
- Appropriate row locking
- Database constraints

> Two administrators approving based on the same outdated timetable: only one valid update commits. The other is **rejected** and requires revalidation.

### 6.6 Prevent Double Booking

- PostgreSQL range types and GiST exclusion constraints for overlapping reservations
- Equivalent validation for faculty and student batch overlaps
- Time-slot overlap checks must work even with different IDs but overlapping clock times

### 6.7 Schedule Versioning

- Never overwrite scheduling history
- Maintain identifiable schedule versions
- Active schedule unchanged during simulation
- Approved update creates a **new** active version

### 6.8 Audit Logging

Record:
- User identity & action
- Timestamp
- Previous and new values
- Schedule version
- Approval details

> Audit records must **not** be silently overwritten.

### 6.9 Transactional Notifications

Use a **transactional outbox pattern**:
- Create notification events inside the same transaction as the approved schedule update
- A separate notification worker delivers them after commit
- Prevents notifications for rolled-back changes

---

## 7. Engine 3 — OS Engine (20%)

> Implement **actual** operating system concepts. Do not simply name classes after OS concepts without demonstrating their behavior.

### 7.1 Multiprocessing Processes

| Process | Responsibilities |
|---|---|
| **Scheduling Coordinator** | Receive requests, create jobs, assign priorities, dispatch, track states |
| **Conflict Detector** | Identify conflicts, analyze resource availability, report affected assignments |
| **Scheduling Worker** | Execute DSA algorithms, generate candidates, validate alternatives, return results |
| **Result Handler** | Process worker results, update job status, store valid results, report errors |
| **Notification Worker** | Retrieve committed events, deliver notifications, record delivery, retry failures |

### 7.2 Priority-Based Job Scheduling

| Job Type | Priority |
|---|---|
| Emergency resource recovery | Highest |
| Urgent conflict resolution | High |
| Routine timetable update | Medium |
| Initial timetable generation | Normal |
| Background analytics | Low |

**Aging:** Waiting jobs gradually increase in priority to prevent starvation.  
**Configuration:** Priority and aging policies must be configurable.

### 7.3 Inter-Process Communication

Flow: `Coordinator → Job Queue → Scheduling Worker → Result Queue → Result Handler`

**Structured job message fields:**
- Job ID, Job type, Priority
- Payload reference, Schedule version
- Creation time, Retry count

> Workers receive **serializable job data** and use their own managed database sessions. Do not pass database connections between processes.

### 7.4 Synchronization

- Mutex locks
- Semaphores
- Process-safe shared state

> Use database transactions for persistent state integrity. Do not rely on process-local mutexes to prevent conflicts between separate application instances.

### 7.5 Deadlock Prevention

Strategies:
- Consistent lock ordering
- Minimal critical sections
- Lock timeouts
- Avoid unnecessary nested locks

> Create a **controlled test** demonstrating a deadlock scenario, then demonstrate the prevention strategy.

### 7.6 Worker Pool

- Configurable number of scheduling workers
- Worker initialization and job assignment
- Worker health monitoring
- Graceful shutdown
- Job timeout and failure detection

### 7.7 Failure Recovery

If a worker crashes:
1. Detect failure
2. Mark job as interrupted
3. Check if safe to retry
4. Requeue eligible jobs
5. Limit retry attempts
6. Record the failure
7. Preserve database consistency

> Use idempotency keys to avoid duplicate effects on retry.

### 7.8 Job States

```
PENDING → QUEUED → RUNNING → COMPLETED
                           → FAILED → RETRYING
                           → CANCELLED
```

---

## 8. Engine 4 — Web & Networks (20%)

### 8.1 User Roles

| Role | Key Capabilities |
|---|---|
| **University Administrator** | Full system management, generate timetables, approve changes, audit logs |
| **Department Coordinator** | Department resource management, report unavailability, generate/review alternatives |
| **Faculty Member** | View personal timetable, notifications, room changes |
| **Student** | View batch timetable, notifications, classroom/lab info |

> Enforce permissions on the **backend**, not just by hiding frontend buttons.

### 8.2 Frontend Pages

| # | Page | Key Features |
|---|---|---|
| 1 | Login | Email, password, validation, error handling |
| 2 | Main Dashboard | Stats overview, active version, pending jobs, conflicts, recent activity |
| 3 | Resource Management | CRUD for departments, faculty, batches, courses, rooms, labs, equipment |
| 4 | Timetable Management | Weekly/day/faculty/batch/room views, conflict visualization |
| 5 | Schedule Generation | Department/semester/year selection, constraint config, progress monitoring |
| 6 | Conflict Center | Conflict type, affected resources/activities, severity, recovery suggestions |
| 7 | Simulation Center | Resource unavailability, affected assignments, alternatives comparison |
| 8 | Alternative Comparison | Side-by-side metrics, changed assignments, objective scores |
| 9 | Approval Center | Simulation details, version validation, approval controls |
| 10 | Audit Logs | User, action, timestamps, previous/new values |
| 11 | Faculty Dashboard | Personal timetable, course details, notifications |
| 12 | Student Dashboard | Batch timetable, room/lab info, notifications |

### 8.3 REST API Endpoints

#### Authentication
```
POST   /api/v1/auth/login
POST   /api/v1/auth/register
GET    /api/v1/auth/me
```

#### Departments
```
GET    /api/v1/departments
POST   /api/v1/departments
PUT    /api/v1/departments/{id}
DELETE /api/v1/departments/{id}
```

#### Resources
```
GET    /api/v1/faculty
POST   /api/v1/faculty
GET    /api/v1/rooms
POST   /api/v1/rooms
GET    /api/v1/laboratories
POST   /api/v1/laboratories
GET    /api/v1/equipment
POST   /api/v1/equipment
```

#### Courses and Batches
```
GET    /api/v1/courses
POST   /api/v1/courses
GET    /api/v1/batches
POST   /api/v1/batches
```

#### Scheduling
```
POST   /api/v1/schedules/generate
GET    /api/v1/schedules
GET    /api/v1/schedules/{id}
GET    /api/v1/schedules/{id}/conflicts
```

#### Jobs
```
GET    /api/v1/jobs/{id}
POST   /api/v1/jobs/{id}/cancel
```

#### Simulation
```
POST   /api/v1/simulations
GET    /api/v1/simulations/{id}
GET    /api/v1/simulations/{id}/alternatives
```

#### Approval
```
POST   /api/v1/simulations/{id}/approve
POST   /api/v1/simulations/{id}/commit
```

#### Notifications
```
GET    /api/v1/notifications
PATCH  /api/v1/notifications/{id}/read
```

#### Audit
```
GET    /api/v1/audit
```

### 8.4 WebSockets

Events handled via WebSocket:
- Scheduling progress
- Job status updates
- Simulation completion
- Worker status
- Approved timetable updates
- Notifications

> Implement authentication for WebSocket connections. Provide reconnection handling on the frontend. Do **not** use WebSockets as a replacement for ordinary REST operations.

---

## 9. Most Important End-to-End Scenario

### Scenario: Laboratory Unavailability

**Initial state:** Lab 3 assigned to multiple classes, timetable active, all assignments valid.

**Event:** Lab 3 becomes unavailable for two days (maintenance).

| Step | Action | Component |
|---|---|---|
| 1 | Admin reports unavailability (start/end date, reason) | Frontend |
| 2 | FastAPI validates and stores unavailability record | Backend + PostgreSQL |
| 3 | Conflict Detector identifies affected assignments | OS Engine |
| 4 | Coordinator creates high-priority recovery job | OS Engine |
| 5 | Scheduling worker runs DSA engine (DSATUR + CSP + Backtracking) | DSA Engine |
| 6 | Save candidate schedules separately; active timetable unchanged | DBMS Engine |
| 7 | Display alternatives with metrics to admin | Frontend |
| 8 | Admin selects alternative; backend verifies authorization | Backend |
| 9 | Validate active timetable matches simulation's base version | DBMS Engine |
| 10 | Transactional commit: apply changes, new version, audit, notifications | DBMS Engine |
| 11 | Notification worker delivers updates to affected faculty/students | OS Engine |
| 12 | Frontend receives WebSocket update, displays new active timetable | Frontend |

> **This complete scenario must be implemented and tested.**

---

## 10. Functional Requirements

| ID | Requirement |
|---|---|
| FR-01 | Resource Management |
| FR-02 | Automatic Schedule Generation |
| FR-03 | Conflict Detection |
| FR-04 | Alternative Resource Generation |
| FR-05 | What-If Simulation |
| FR-06 | Schedule Approval and Transactional Update |
| FR-07 | Database Management |
| FR-08 | Notifications and Audit Logs |

Each requirement must be connected to a **working feature and tested**.

---

## 11. Non-Functional Requirements

| ID | Requirement | Target |
|---|---|---|
| NFR-01 | Performance | Valid schedule for 100 activities within 10 seconds |
| NFR-02 | Scalability | Process 500 activities without hard-constraint violations |
| NFR-03 | Reliability | Zero invalid schedules committed |
| NFR-04 | Security | Reject unauthorized approval attempts |
| NFR-05 | Data Integrity | Database consistency after failed transactions |
| NFR-06 | Usability | Professional, accessible interface |
| NFR-07 | Maintainability | Modular, documented codebase |
| NFR-08 | Responsiveness | P95 read-only API response < 500ms |

> Do **not** claim these targets are achieved until benchmarked.

---

## 12. Security Requirements

- Secure password hashing (Argon2 or bcrypt)
- JWT authentication
- Backend RBAC enforcement
- Input validation via Pydantic
- SQL injection protection (ORM parameterized queries)
- CORS configuration
- Rate limiting for sensitive endpoints
- Secure environment-variable handling
- Protection against unauthorized schedule modifications
- WebSocket authentication
- Audit records for sensitive actions

> Never store plaintext passwords. Never commit secrets to version control.

---

## 13. Testing Requirements

### DSA Testing
- Graph construction & conflict edge detection
- DSATUR vertex selection
- CSP domain creation, Forward Checking, Backtracking
- Hard & soft constraint validation
- Incremental rescheduling, alternative ranking
- Edge cases: no rooms, no slots, insufficient capacity, faculty unavailable, no feasible schedule

### DBMS Testing
- Foreign key & unique constraints
- Room/faculty overlap prevention
- Transaction rollback
- Schedule version validation
- Concurrent update handling
- Audit record creation

### OS Testing
- Multiple workers, job queue ordering
- Priority handling & aging
- IPC, mutex, semaphore behavior
- Worker crash & retry limits
- Graceful shutdown

### Web Testing
- Login, authentication, RBAC
- Resource CRUD
- Timetable retrieval
- Schedule generation → simulation → approval → commit
- Notifications & WebSocket updates

---

## 14. User Experience and Design Requirements

**Design Direction:** Modern academic administration dashboard

Requirements:
- Clean and consistent typography
- Responsive layouts (desktop, tablet, mobile)
- Clear visual hierarchy
- Professional navigation sidebar
- Consistent spacing and accessible contrast
- Useful empty states and loading indicators
- Error messages and confirmation dialogs
- Toast notifications

**Timetable visual distinctions:**

| State | Visual Treatment |
|---|---|
| Available resource | Default / green indicator |
| Unavailable resource | Grey / strikethrough |
| Conflicting assignment | Red highlight |
| Proposed assignment | Yellow / amber |
| Approved assignment | Blue / success |
| Active timetable version | Prominent badge |

---

## 15. Project Folder Structure

```
AURO/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── layouts/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── context/
│   │   ├── types/
│   │   ├── utils/
│   │   ├── routes/
│   │   └── styles/
│   ├── public/
│   ├── package.json
│   ├── tsconfig.json
│   └── vite.config.ts
│
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── api/
│   │   │   └── routes/
│   │   │       ├── auth.py
│   │   │       ├── departments.py
│   │   │       ├── faculty.py
│   │   │       ├── rooms.py
│   │   │       ├── laboratories.py
│   │   │       ├── equipment.py
│   │   │       ├── courses.py
│   │   │       ├── batches.py
│   │   │       ├── schedules.py
│   │   │       ├── simulations.py
│   │   │       ├── approvals.py
│   │   │       ├── notifications.py
│   │   │       ├── audit.py
│   │   │       └── jobs.py
│   │   ├── core/
│   │   │   ├── config.py
│   │   │   ├── security.py
│   │   │   └── permissions.py
│   │   ├── db/
│   │   │   ├── database.py
│   │   │   ├── models/
│   │   │   ├── migrations/
│   │   │   └── repositories/
│   │   ├── schemas/
│   │   ├── services/
│   │   │   ├── resource_service.py
│   │   │   ├── schedule_service.py
│   │   │   ├── simulation_service.py
│   │   │   ├── approval_service.py
│   │   │   └── notification_service.py
│   │   ├── dsa/
│   │   │   ├── graph.py
│   │   │   ├── dsatur.py
│   │   │   ├── csp.py
│   │   │   ├── backtracking.py
│   │   │   ├── forward_checking.py
│   │   │   ├── priority_queue.py
│   │   │   ├── hash_maps.py
│   │   │   ├── constraint_validator.py
│   │   │   ├── optimizer.py
│   │   │   ├── conflict_detector.py
│   │   │   └── incremental_scheduler.py
│   │   ├── os_engine/
│   │   │   ├── coordinator.py
│   │   │   ├── worker.py
│   │   │   ├── worker_pool.py
│   │   │   ├── job_queue.py
│   │   │   ├── ipc.py
│   │   │   ├── synchronization.py
│   │   │   ├── recovery.py
│   │   │   └── notification_worker.py
│   │   ├── websocket/
│   │   │   ├── manager.py
│   │   │   └── events.py
│   │   └── tests/
│   │       ├── unit/
│   │       ├── integration/
│   │       └── performance/
│   ├── requirements.txt
│   └── Dockerfile
│
├── docker-compose.yml
├── docs/
│   ├── MASTER_PROMPT.md       ← this file
│   ├── architecture.md
│   ├── database_schema.md
│   ├── api_documentation.md
│   ├── algorithms.md
│   └── testing.md
├── .env.example
└── README.md
```

---

## 16. Development Strategy

| Phase | Focus | Deliverable |
|---|---|---|
| **Phase 1** | Project Foundation | Structure, Docker, DB, migrations, all services start |
| **Phase 2** | Auth & Resource Management | Users, roles, CRUD for all resources |
| **Phase 3** | DSA Engine | Graph, DSATUR, CSP, Backtracking, Forward Checking |
| **Phase 4** | Initial Schedule Generation | Activities → graph → CSP → assignments → persist |
| **Phase 5** | OS Engine | Coordinator, workers, IPC, priorities, aging, pool |
| **Phase 6** | Simulation & Incremental Recovery | Unavailability → alternatives → save separately |
| **Phase 7** | Approval & DB Safety | Versioning, locking, transactional commit, audit |
| **Phase 8** | Frontend | All pages connected to real backend |
| **Phase 9** | Real-Time Communication | WebSockets, job progress, notifications |
| **Phase 10** | Testing & Deployment | All tests, E2E demo, Docker images, documentation |

> After completing each phase: show files created, explain implementation, run tests, report and fix errors, then verify before proceeding.

---

## 17. Sample Data and Demonstration

### Seed Data Requirements

| Entity | Count |
|---|---|
| Departments | 2 |
| Faculty members | 10 |
| Student batches | 5 |
| Courses | 10 |
| Classrooms | 5 |
| Laboratories | 3 |
| Equipment records | Multiple |
| Time slots | Full weekly schedule |
| Faculty availability records | All faculty × all slots |

### Demonstration Scenario

**Lab 3 becomes unavailable for two days.**

The application must:
1. Detect affected assignments
2. Find compatible alternatives
3. Generate feasible schedules
4. Compare alternatives
5. Preserve active timetable during simulation
6. Allow administrator approval
7. Commit a new schedule version
8. Notify affected users

### Demo Credentials

> See `docs/DEMO_CREDENTIALS.md` *(development documentation only — never commit production credentials)*

---

## 18. Documentation Requirements

### README Contents
- Project overview and problem statement
- Features and architecture
- Four-engine explanation
- Technology stack with versions
- Installation, configuration, migration, seed instructions
- Frontend, backend, and Docker startup
- API documentation
- Testing instructions
- Demo scenario walkthrough

### Additional Documentation
- DSA algorithm explanations (`docs/algorithms.md`)
- Database ER diagram (`docs/database_schema.md`)
- System architecture diagram (`docs/architecture.md`)
- API structure (`docs/api_documentation.md`)
- Testing guide (`docs/testing.md`)
- Use-case diagram
- Process communication diagram
- Scheduling workflow
- Transaction and recovery workflow

---

## 19. Strict Implementation Rules

1. ❌ Do not create a frontend-only prototype
2. ❌ Do not hardcode timetable outputs
3. ❌ Do not use random scheduling as a substitute for DSA algorithms
4. ❌ Do not ignore hard constraints
5. ❌ Do not modify the active timetable during simulation
6. ❌ Do not allow approval without authorization
7. ❌ Do not commit an outdated simulation without version validation
8. ❌ Do not claim multiprocessing when tasks are executed sequentially
9. ❌ Do not claim IPC without actual process communication
10. ❌ Do not claim deadlock prevention without implementing and testing a strategy
11. ❌ Do not claim database transactions without actual transactional operations
12. ❌ Do not create notifications before a successful commit
13. ❌ Do not leave buttons disconnected from backend functionality
14. ❌ Do not silently ignore errors
15. ❌ Do not swallow exceptions
16. ❌ Do not expose sensitive information in API responses
17. ❌ Do not store passwords in plaintext
18. ❌ Do not use SQLite as a replacement for PostgreSQL
19. ❌ Do not replace DSATUR or CSP with a generic greedy algorithm
20. ❌ Do not implement the entire system as one enormous Python file
21. ❌ Do not claim performance targets without benchmark results
22. ❌ Do not mark a phase complete until its tests pass

> If a feature cannot be implemented correctly, **explain the limitation and propose a technically sound alternative** instead of pretending it works.

---

## 20. Final Acceptance Criteria

### Authentication & Authorization
- [ ] User authentication
- [ ] Role-based authorization

### Resource Management
- [ ] Department management
- [ ] Faculty management
- [ ] Student batch management
- [ ] Course management
- [ ] Classroom management
- [ ] Laboratory management
- [ ] Equipment management
- [ ] Availability management

### DSA Engine
- [ ] Initial timetable generation
- [ ] Conflict graph construction
- [ ] DSATUR implementation
- [ ] CSP implementation
- [ ] Backtracking implementation
- [ ] Forward Checking implementation
- [ ] Priority Queue implementation
- [ ] Hash Map usage
- [ ] Hard constraint validation
- [ ] Soft constraint optimization

### Scheduling Intelligence
- [ ] Conflict detection
- [ ] Incremental schedule recovery
- [ ] Alternative schedule generation
- [ ] What-if simulation
- [ ] Active schedule preservation during simulation
- [ ] Alternative comparison

### Database Safety
- [ ] Administrator approval flow
- [ ] Schedule versioning
- [ ] Transactional schedule commit
- [ ] Transaction rollback
- [ ] Concurrent update protection

### OS Engine
- [ ] Multiprocessing
- [ ] Inter-process communication
- [ ] Priority-based job scheduling
- [ ] Aging
- [ ] Worker Pool
- [ ] Synchronization
- [ ] Worker failure recovery

### Web & Communication
- [ ] REST APIs
- [ ] WebSockets
- [ ] Notifications
- [ ] Audit logs
- [ ] Responsive dashboards

### Quality & Deployment
- [ ] Automated tests
- [ ] Docker deployment
- [ ] Complete documentation
- [ ] End-to-end Lab 3 unavailability demonstration

---

*Document maintained by the AURO development team. Update this file when the specification changes and increment the version number.*

