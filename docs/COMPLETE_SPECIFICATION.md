# AURO — Complete Project Specification
## Autonomous University Resource Orchestrator

> **Document Type:** Finalized Complete Specification — Single Source of Truth  
> **Version:** 2.0 (Supersedes MASTER_PROMPT.md v1.0)  
> **Status:** ✅ FINALIZED — Use this as the authoritative project definition  
> **Last Updated:** 2026-11-06

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Proposed Solution](#3-proposed-solution)
4. [Project Scope](#4-project-scope)
5. [Four-Engine Architecture](#5-four-engine-architecture)
6. [Engine 1 — DSA (35%)](#6-engine-1--dsa-35)
7. [Engine 2 — DBMS (25%)](#7-engine-2--dbms-25)
8. [Engine 3 — OS (20%)](#8-engine-3--operating-systems-20)
9. [Engine 4 — Web & Networks (20%)](#9-engine-4--web-technologies-and-networks-20)
10. [End-to-End Workflow](#10-most-important-end-to-end-workflow)
11. [Two Primary Scheduling Workflows](#11-two-primary-scheduling-workflows)
12. [Scheduling Mathematical Model](#12-scheduling-mathematical-model)
13. [Functional Requirements](#13-functional-requirements)
14. [Non-Functional Requirements](#14-non-functional-requirements)
15. [Use Cases](#15-use-cases)
16. [User Stories](#16-user-stories)
17. [Database Entity Relationships](#17-database-entity-relationships)
18. [Key Performance Indicators](#18-key-performance-indicators)
19. [Security Requirements](#19-security-requirements)
20. [Testing Strategy](#20-testing-strategy)
21. [Frontend Design Requirements](#21-frontend-design-requirements)
22. [Project Folder Structure](#22-project-folder-structure)
23. [Development Phases](#23-development-phases)
24. [Sample Data](#24-sample-data)
25. [Documentation](#25-documentation)
26. [Strict Implementation Rules](#26-strict-implementation-rules)
27. [Final Acceptance Checklist](#27-final-acceptance-checklist)
28. [Final Project Definition](#28-final-project-definition)
29. [Execution Instructions](#29-execution-instructions)

---

## 1. Project Overview

AURO is an **intelligent university resource orchestration system** designed to automate academic timetable generation, manage institutional resources, detect scheduling conflicts and recover from unexpected resource disruptions.

Universities manage multiple interconnected resources:

| Resource | Description |
|---|---|
| Classrooms | Physical rooms for lectures |
| Laboratories | Specialized rooms with equipment |
| Faculty members | Teaching staff |
| Student batches | Groups of students per semester |
| Courses and subjects | Academic offerings |
| Laboratory equipment | Physical apparatus assigned to labs |
| Time slots | Day and time periods |
| Academic activities | Scheduled class sessions |

### Why these resources are complex to manage

- A classroom cannot accommodate two classes simultaneously.
- A faculty member cannot teach two classes at the same time.
- A student batch cannot attend two lectures simultaneously.
- A laboratory must have the equipment required for its assigned practical session.
- Room capacity must be sufficient for the assigned batch.

Manual management becomes increasingly difficult as the university grows.

### AURO solves this using four integrated engineering domains

| Domain | Weight | Responsibility |
|---|---:|---|
| Data Structures & Algorithms (DSA) | 35% | Intelligent scheduling and optimization |
| Database Management Systems (DBMS) | 25% | Persistent storage and data integrity |
| Operating Systems (OS) | 20% | Process coordination and recovery |
| Web Technologies & Computer Networks | 20% | User interface and communication |

> These percentages represent the **planned academic contribution** of each engine.

---

## 2. Problem Statement

Universities frequently depend on manual or partially automated systems for timetable preparation and resource allocation.

**Problems with existing systems:**

1. Multiple activities may be assigned to the same classroom simultaneously.
2. Faculty members may receive overlapping teaching assignments.
3. Student batches may have conflicting classes.
4. Laboratories may not have the required equipment.
5. Unexpected resource unavailability can disrupt existing schedules.
6. Administrators must manually find alternative classrooms and time slots.
7. Changes to one assignment may introduce additional conflicts.
8. Existing systems may not provide proper schedule simulation and comparison.
9. There may be no reliable mechanism to preserve schedule history.
10. Administrators may unintentionally introduce inconsistent timetable changes.

**The need:** An intelligent system that can automatically generate valid timetables, detect conflicts, identify alternative resources and recover from scheduling disruptions while minimizing changes to unaffected assignments.

---

## 3. Proposed Solution

AURO provides a centralized university resource management and intelligent scheduling platform.

**The system must:**

1. Manage university resources
2. Generate an initial timetable
3. Store the timetable in PostgreSQL
4. Detect resource and scheduling conflicts
5. Identify assignments affected by resource unavailability
6. Generate alternative schedules
7. Simulate proposed changes without modifying the active timetable
8. Compare alternative schedules
9. Allow authorized administrators to approve changes
10. Safely commit approved changes
11. Notify affected faculty members and students
12. Maintain audit logs and schedule versions

### Two main operating modes

#### Mode A: Initial timetable generation

Generate a complete timetable at the beginning of a semester or when a new academic schedule is required.

#### Mode B: Incremental schedule recovery

When an existing resource becomes unavailable:
- Identify affected assignments
- Reschedule **only the necessary activities** wherever possible
- **Preserve unaffected assignments**
- If incremental recovery is impossible, **report unresolved conflicts** rather than generating an invalid timetable

### Main innovation

> AURO is not just a timetable generator.
>
> Its defining capability is: **Intelligent schedule recovery with what-if simulation, alternative comparison, approval-based updates and minimal disruption to existing academic operations.**

---

## 4. Project Scope

### Resources managed

| Resource | Information |
|---|---|
| Departments | Department details |
| Faculty | Faculty information, availability and workload |
| Student batches | Batch, semester and student count |
| Courses | Course code, name and requirements |
| Classrooms | Capacity, building and availability |
| Laboratories | Type, capacity and equipment |
| Equipment | Equipment type, quantity and availability |
| Time slots | Day, start time and end time |
| Course offerings | Course, faculty and batch relationships |
| Assignments | Actual timetable allocations |

### User roles and capabilities

#### University Administrator
- Manage all resources
- Generate timetables
- Configure scheduling constraints
- Detect conflicts
- Create simulations
- Compare alternatives
- Approve changes
- Commit updates
- View audit logs

#### Department Coordinator
- Manage department resources
- View department timetables
- Report resource unavailability
- Review scheduling conflicts
- Review alternatives (per permissions)

#### Faculty Member
- View personal timetable
- View course and classroom details
- Receive timetable notifications

#### Student
- View batch timetable
- View classroom and laboratory information
- Receive published timetable updates

> **Rule:** Enforce authorization in the **backend**, not merely by hiding frontend controls.

---

## 5. Four-Engine Architecture

```
                UNIVERSITY USERS
                       │
        ┌──────────────┼──────────────┐
        │              │              │
   Administrator  Faculty         Students
        │              │              │
        └──────────────┼──────────────┘
                       │
                 REACT FRONTEND
                       │
                 FASTAPI BACKEND
                       │
             APPLICATION COORDINATOR
                       │
       ┌───────────────┼────────────────┐
       │               │                │
   DSA ENGINE       OS ENGINE        DBMS ENGINE
       │               │                │
   Scheduling      Job Manager       PostgreSQL
   Conflict        Multiprocessing   Transactions
   Optimization    Worker Pool       Constraints
   Recovery        IPC               Versioning
       │               │                │
       └───────────────┼────────────────┘
                       │
               REST / WEBSOCKETS
                       │
               RESULTS AND UPDATES
                       │
                   FRONTEND
```

### Engine responsibility summary

| Engine | Main Responsibility |
|---|---|
| DSA | Determine valid and optimized schedules |
| DBMS | Store and protect university scheduling information |
| OS | Coordinate and execute scheduling jobs |
| Web & Networks | Provide user interaction and communication |

---

## 6. Engine 1 — DSA (35%)

The DSA engine is responsible for pre-built timetable validation, conflict detection, BFS impact analysis, priority-based recovery task dispatching, and minimum-disruption schedule recovery using CSP with Backtracking and Forward Checking.

### Finalized 7 DSA Components

1. **Conflict Graph** (Resource-sharing relationships between activities)
2. **BFS (Breadth-First Search)** (Explores connected affected activities)
3. **Priority Queue (Min-Heap)** (Orders recovery jobs by urgency)
4. **Hash Maps** (O(1) in-memory fast lookups of resources and assignments)
5. **Constraint Satisfaction Problem (CSP)** (Models the recovery subproblem)
6. **Backtracking** (Recursive candidate assignment search and state restoration)
7. **Forward Checking** (Constraint propagation and domain pruning)

> **Exclusion Notice:** 
> - **DSATUR** is excluded because AURO uses pre-built university timetables rather than generating an initial timetable from an empty graph.
> - **DFS** is excluded in favor of BFS for level-by-level impact analysis.

---

### 6.1 Graph Theory — Conflict Graph

Represent scheduling activities using a graph.

- **Vertices** = class sessions or scheduling activities
- **Edges** = incompatibilities (activities that cannot occur simultaneously)

**Edge conditions (shared exclusive resources):**

```
Activity A: Faculty 1, Batch A
Activity B: Faculty 1, Batch B  → edge (shared Faculty 1)
Activity C: Faculty 2, Batch A  → edge with A (shared Batch A)
                                → NO edge with B
```

**Required graph operations:**

| Operation | Description |
|---|---|
| Add vertex | Register a new activity |
| Remove vertex | Deregister an activity |
| Add edge | Mark two activities as conflicting |
| Remove edge | Remove a conflict relationship |
| Retrieve neighbors | Get all conflicting activities |
| Calculate degree | Count conflicts for an activity |
| Check adjacency | Test if two activities conflict |
| Generate conflict info | Get reason for the conflict |

> Construct the graph **dynamically from actual university scheduling data**. Do not hardcode conflicts.

> **Important:** Not every scheduling constraint should be a graph edge. Room capacity, equipment requirements, availability and complex time constraints must also be validated through CSP and the constraint validator.

---

### 6.2 BFS (Breadth-First Search)

BFS is AURO's single graph traversal method for impact analysis.

When a resource or activity is disrupted (e.g. Lab outage or faculty absence):
1. Start at the directly affected activity vertex in the Conflict Graph.
2. Traverse adjacent edges level by level using a FIFO Queue.
3. Identify the connected cluster of potentially impacted activities.
4. Establish the exact scope for the recovery subproblem without disturbing unaffected cohorts.

- **Time Complexity:** $O(V + E)$
- **Space Complexity:** $O(V)$

---

### 6.3 Constraint Satisfaction Problem (CSP)

Model timetable generation as a CSP.

**Variables:** Scheduling activities or class sessions

**Domains:** Feasible room-time combinations for each activity

**Constraints:**

#### Hard Constraints *(must NEVER be violated)*

| Constraint | Description |
|---|---|
| Room exclusivity | No room double-booked |
| Faculty exclusivity | No faculty teaching two simultaneous classes |
| Batch exclusivity | No batch attending two simultaneous classes |
| Room capacity | Room must hold the batch size |
| Equipment requirements | Required equipment must be available |
| Faculty availability | Faculty must be available in assigned slot |
| Room availability | Room must not be marked unavailable |
| Activity duration | Session must fit within the slot |
| Laboratory requirements | Lab courses need lab rooms |
| Resource unavailability | Unavailable resources cannot be assigned |

#### Soft Constraints *(should be optimized)*

| Constraint | Description |
|---|---|
| Faculty preferences | Preferred time slots honoured |
| Room preferences | Preferred rooms used when possible |
| Reduced gaps | Minimize idle time between classes |
| Balanced workloads | Even distribution across days |
| Better utilization | Rooms well-utilized |
| Minimum disruption | Fewest changes from existing schedule |

> **Rule:** Hard constraints **validated independently** and must **never** be traded away for a better soft-constraint score.

---

### 6.4 Backtracking

Backtracking searches for valid assignments.

```
1. Select next unassigned activity
2. Assign a candidate (room, time slot)
3. Check hard constraints
4. If valid → continue to next activity
5. If a later activity becomes impossible → backtrack
6. Reverse the assignment
7. Try the next candidate
8. If no candidates remain → backtrack further
```

Implement **proper restoration** of domains and assignment state when backtracking.

---

### 6.5 Forward Checking

Forward Checking improves backtracking efficiency.

**After assigning an activity:**

1. Examine remaining unassigned activities
2. Remove incompatible values from their domains
3. Check whether any domain becomes empty
4. If yes → backtrack **immediately** (before attempting to continue)

This eliminates unnecessary search branches early.

---

### 6.6 Priority Queue

A supporting data structure used for:

- Managing urgent recovery operations
- Prioritizing classes by proximity of start time
- Ordering background tasks
- Priority Queue is a **supporting structure**, not a separate scheduling algorithm

---

### 6.7 Hash Maps

Use hash maps for efficient O(1) key-based lookups.

```
Faculty ID     → Faculty information
Room ID        → Room information
Batch ID       → Batch information
Course ID      → Course information
Activity ID    → Scheduling state
Resource ID    → Availability
```

> Use hash maps to **reduce repeated searches** during scheduling.

---

### 6.8 Scheduling Objective

**Stage A: Feasibility** — Find assignments satisfying every hard constraint.

**Stage B: Optimization** — Rank valid assignments using a weighted soft-constraint objective.

$$C = w_1 D + w_2 P + w_3 G + w_4 U$$

| Variable | Meaning |
|---|---|
| $D$ | Number of changed assignments |
| $P$ | Preference violations |
| $G$ | Unnecessary gaps |
| $U$ | Resource utilization penalty |
| $w_1, w_2, w_3, w_4$ | Configurable weights |

> Hard constraints validated **separately**. Never traded for a better objective score.

---

### 6.9 Incremental Rescheduling

**One of the most important DSA features.**

When a resource becomes unavailable:

```
1. Identify assignments using the affected resource
2. Identify directly affected activities
3. Expand the affected set if alternatives introduce new conflicts
4. Build the affected CSP subproblem
5. Keep unaffected assignments fixed wherever possible
6. Generate alternatives
7. Minimize the number of modified assignments
8. Validate the complete resulting timetable
```

> If no valid solution exists → **return unresolved conflicts**.  
> Do **not** silently generate an invalid schedule.

---

### 6.10 Alternative Generation

Generate multiple feasible candidate schedules where possible.

**Each alternative must include:**

| Field | Description |
|---|---|
| Changed assignments | What was rescheduled |
| Unchanged assignments | What was preserved |
| New room allocations | Replacement rooms |
| New time-slot allocations | Replacement slots |
| Unresolved conflicts | Any remaining issues |
| Soft-constraint score | Optimization quality |
| Number of modified activities | Disruption measure |
| Resource utilization | Efficiency metric |

Rank alternatives using configurable optimization criteria.

### Revised DSA Workflow

```text
                  PRE-BUILT TIMETABLES
                           |
                           v
                 IMPORT AND VALIDATION
                           |
                           v
                LOAD ACTIVE SCHEDULE VERSION
                           |
                           v
                 BUILD RESOURCE INDEXES
                     (HASH MAPS)
                           |
                           v
                  BUILD CONFLICT GRAPH
                           |
                           v
                RESOURCE CHANGE / INCIDENT
                           |
                           v
               IDENTIFY AFFECTED ACTIVITIES
                           |
                           v
                  BFS GRAPH EXPLORATION
                           |
                           v
                PRIORITIZE RECOVERY JOBS
                    (PRIORITY QUEUE)
                           |
                           v
                    BUILD RECOVERY CSP
                           |
                           v
                      BACKTRACKING
                           |
                           v
                    FORWARD CHECKING
                           |
                           v
                 GENERATE FEASIBLE OPTIONS
                           |
                           v
                VALIDATE COMPLETE CANDIDATE
                           |
                           v
              COMPARE DISRUPTION / SIMULATE
                           |
                           v
                  ADMINISTRATOR APPROVAL
                           |
                           v
                  COMMIT NEW VERSION
```

---

## 7. Engine 2 — DBMS (25%)

### Purpose

Manage persistent university data, timetable storage, transactions, concurrency control, database constraints and schedule history.

### Finalized DBMS Components

1. PostgreSQL
2. Database Normalization
3. Advanced Indexing
4. ACID Transactions
5. Concurrency Control
6. Database Constraints
7. Schedule Versioning
8. Audit Logging
9. Transactional Notification Outbox

---

### 7.1 PostgreSQL

**Primary relational database.** Do **not** use MongoDB or SQLite.

**Reasons:**
- Interconnected resource relationships require foreign keys and joins
- Schedule commitments require ACID transactions
- Conflict prevention requires database-level exclusion constraints
- History requires reliable versioning

---

### 7.2 Database Normalization

Design appropriately normalized relational tables.

- Avoid unnecessary duplication
- Use foreign-key relationships
- Design for query efficiency

---

### 7.3 Indexing Strategy

**Index types to use:**

| Type | Use Case |
|---|---|
| B-tree | Standard single-column lookups |
| Composite | Multi-column filter queries |
| Partial | Subset queries (e.g., `WHERE is_active = TRUE`) |

**Common index targets:**
`faculty_id`, `room_id`, `batch_id`, `course_id`, `slot_id`, `schedule_version_id`, `simulation_id`

> Use `EXPLAIN ANALYZE` to investigate important queries.

---

### 7.4 ACID Transactions

**Schedule approval transaction flow:**

```
1. BEGIN TRANSACTION
2. Verify current schedule version
3. Validate proposed assignments (hard constraints)
4. Apply approved changes
5. Create new ScheduleVersion (ACTIVE)
6. Deactivate previous version
7. Create AuditLog records
8. Create NotificationOutbox records
9. COMMIT

If any critical step fails → ROLLBACK (entire transaction)
```

---

### 7.5 Concurrency Control

**Two-tier protection:**

| Mechanism | Purpose |
|---|---|
| Optimistic locking | Version-based change validation |
| Pessimistic locking | `SELECT FOR UPDATE` during approval |
| Schedule-version validation | Reject stale simulation commits |
| Database constraints | Last line of defence against invalid data |

**Scenario:** Two administrators submit changes based on the same outdated schedule version → **only the first valid commit succeeds**. The second receives a conflict error and must revalidate.

---

### 7.6 Database Constraints

| Constraint Type | Examples |
|---|---|
| Primary keys | All entity IDs |
| Foreign keys | All relational references |
| Unique constraints | Email, department code, room name |
| Check constraints | `capacity > 0`, `end_time > start_time`, role enum |
| Exclusion constraints | Prevent overlapping room reservations (GiST) |

> **Important:** Different time-slot IDs may overlap in clock time. The system must check **actual time intervals**, not only slot IDs.
>
> Example: 9:00–10:00 and 9:30–10:30 **overlap** even if they have different slot IDs.

---

### 7.7 Schedule Versioning

```
Schedule Version 1 (archived)
        │
        ▼
Schedule Version 2 (superseded)
        │
        ▼
Schedule Version 3 (active)
```

- Every approved update creates a **new version**
- History is **never overwritten**
- Simulations must reference the base version from which they were generated
- The active schedule remains unchanged during simulation

---

### 7.8 Audit Logging

Record for every important action:

| Field | Description |
|---|---|
| User | Who performed the action |
| Action | What was done |
| Timestamp | When it happened |
| Previous values | JSONB snapshot before change |
| New values | JSONB snapshot after change |
| Schedule version | Which version was affected |
| Approval information | Authorization details |

---

### 7.9 Transactional Outbox

Create notification events **inside the same transaction** as the approved schedule update.

```
TRANSACTION:
├── Apply schedule changes
├── Create new ScheduleVersion
├── Create AuditLog entries
└── Create NotificationOutbox entries  ← same transaction

AFTER COMMIT:
└── Notification Worker polls outbox → delivers notifications
```

> This prevents notifications for changes that were **rolled back**.

---

### 7.10 Complete Database Schema

#### Department

```sql
department_id    UUID PRIMARY KEY
department_name  VARCHAR(255) NOT NULL
department_code  VARCHAR(20)  NOT NULL UNIQUE
created_at       TIMESTAMPTZ  NOT NULL DEFAULT now()
```

#### User

```sql
user_id       UUID PRIMARY KEY
full_name     VARCHAR(255) NOT NULL
email         VARCHAR(255) NOT NULL UNIQUE
password_hash TEXT         NOT NULL
role          VARCHAR(50)  NOT NULL  -- admin|coordinator|faculty|student
department_id UUID REFERENCES department(department_id)
is_active     BOOLEAN      NOT NULL DEFAULT TRUE
created_at    TIMESTAMPTZ  NOT NULL DEFAULT now()
```

#### Faculty

```sql
faculty_id          UUID PRIMARY KEY
user_id             UUID NOT NULL UNIQUE REFERENCES user(user_id)
department_id       UUID NOT NULL REFERENCES department(department_id)
designation         VARCHAR(100)
maximum_weekly_load INTEGER NOT NULL DEFAULT 20
```

#### StudentBatch

```sql
batch_id      UUID PRIMARY KEY
batch_name    VARCHAR(100) NOT NULL
department_id UUID NOT NULL REFERENCES department(department_id)
semester      INTEGER NOT NULL CHECK (semester BETWEEN 1 AND 10)
student_count INTEGER NOT NULL CHECK (student_count > 0)
academic_year VARCHAR(9)   NOT NULL  -- e.g. '2025-2026'
```

#### Course

```sql
course_id         UUID PRIMARY KEY
course_code       VARCHAR(20)  NOT NULL UNIQUE
course_name       VARCHAR(255) NOT NULL
department_id     UUID NOT NULL REFERENCES department(department_id)
course_type       VARCHAR(50)  NOT NULL  -- lecture|lab|tutorial|seminar
required_duration INTEGER      NOT NULL  -- minutes per session
requires_lab      BOOLEAN      NOT NULL DEFAULT FALSE
```

#### Room

```sql
room_id   UUID PRIMARY KEY
room_name VARCHAR(100) NOT NULL
building  VARCHAR(100)
capacity  INTEGER NOT NULL CHECK (capacity > 0)
room_type VARCHAR(50) NOT NULL  -- classroom|lab|seminar_hall|auditorium
is_active BOOLEAN     NOT NULL DEFAULT TRUE
```

#### Laboratory

```sql
laboratory_id   UUID PRIMARY KEY
room_id         UUID NOT NULL UNIQUE REFERENCES room(room_id)
laboratory_name VARCHAR(100) NOT NULL
laboratory_type VARCHAR(100)  -- computer|physics|chemistry|network|...
```

#### Equipment

```sql
equipment_id   UUID PRIMARY KEY
equipment_name VARCHAR(255) NOT NULL
equipment_type VARCHAR(100)
quantity       INTEGER NOT NULL DEFAULT 1 CHECK (quantity >= 0)
```

#### RoomEquipment *(Junction)*

```sql
room_id            UUID NOT NULL REFERENCES room(room_id)
equipment_id       UUID NOT NULL REFERENCES equipment(equipment_id)
available_quantity INTEGER NOT NULL DEFAULT 0
PRIMARY KEY (room_id, equipment_id)
```

#### TimeSlot

```sql
slot_id     UUID PRIMARY KEY
day_of_week INTEGER NOT NULL CHECK (day_of_week BETWEEN 0 AND 6)
start_time  TIME NOT NULL
end_time    TIME NOT NULL
CHECK (end_time > start_time)
UNIQUE (day_of_week, start_time, end_time)
```

#### CourseOffering

```sql
offering_id      UUID    PRIMARY KEY
course_id        UUID    NOT NULL REFERENCES course(course_id)
faculty_id       UUID    NOT NULL REFERENCES faculty(faculty_id)
batch_id         UUID    NOT NULL REFERENCES student_batch(batch_id)
sessions_per_week INTEGER NOT NULL DEFAULT 1
UNIQUE (course_id, faculty_id, batch_id)
```

#### FacultyAvailability

```sql
availability_id UUID PRIMARY KEY
faculty_id      UUID    NOT NULL REFERENCES faculty(faculty_id)
slot_id         UUID    NOT NULL REFERENCES time_slot(slot_id)
is_available    BOOLEAN NOT NULL DEFAULT TRUE
UNIQUE (faculty_id, slot_id)
```

#### ResourceUnavailability

```sql
unavailability_id UUID PRIMARY KEY
resource_type     VARCHAR(50) NOT NULL  -- faculty|room|laboratory|equipment
resource_id       UUID NOT NULL
start_datetime    TIMESTAMPTZ NOT NULL
end_datetime      TIMESTAMPTZ NOT NULL
reason            TEXT
created_by        UUID NOT NULL REFERENCES user(user_id)
created_at        TIMESTAMPTZ NOT NULL DEFAULT now()
CHECK (end_datetime > start_datetime)
```

#### ScheduleVersion

```sql
schedule_version_id UUID PRIMARY KEY
version_number      INTEGER     NOT NULL
status              VARCHAR(50) NOT NULL  -- draft|active|archived|superseded
created_by          UUID        NOT NULL REFERENCES user(user_id)
created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
approved_at         TIMESTAMPTZ
description         TEXT
```

#### Assignment

```sql
assignment_id       UUID PRIMARY KEY
schedule_version_id UUID NOT NULL REFERENCES schedule_version(schedule_version_id)
offering_id         UUID NOT NULL REFERENCES course_offering(offering_id)
room_id             UUID NOT NULL REFERENCES room(room_id)
slot_id             UUID NOT NULL REFERENCES time_slot(slot_id)
session_date_or_day DATE
status              VARCHAR(50) NOT NULL DEFAULT 'assigned'
                    -- assigned|cancelled|rescheduled
UNIQUE (schedule_version_id, room_id, slot_id, session_date_or_day)
```

#### Simulation

```sql
simulation_id            UUID PRIMARY KEY
base_schedule_version_id UUID NOT NULL REFERENCES schedule_version(schedule_version_id)
trigger_unavailability_id UUID REFERENCES resource_unavailability(unavailability_id)
status                   VARCHAR(50) NOT NULL  -- pending|running|completed|failed|approved
objective_score          FLOAT
created_by               UUID NOT NULL REFERENCES user(user_id)
created_at               TIMESTAMPTZ NOT NULL DEFAULT now()
completed_at             TIMESTAMPTZ
```

#### SimulationAssignment

```sql
simulation_assignment_id UUID PRIMARY KEY
simulation_id            UUID NOT NULL REFERENCES simulation(simulation_id)
offering_id              UUID NOT NULL REFERENCES course_offering(offering_id)
proposed_room_id         UUID REFERENCES room(room_id)
proposed_slot_id         UUID REFERENCES time_slot(slot_id)
is_changed               BOOLEAN NOT NULL DEFAULT FALSE
is_conflict              BOOLEAN NOT NULL DEFAULT FALSE
conflict_reason          TEXT
```

#### AuditLog

```sql
audit_id            UUID PRIMARY KEY
user_id             UUID NOT NULL REFERENCES user(user_id)
action              VARCHAR(100) NOT NULL
entity_type         VARCHAR(100) NOT NULL
entity_id           UUID
previous_values     JSONB
new_values          JSONB
schedule_version_id UUID REFERENCES schedule_version(schedule_version_id)
timestamp           TIMESTAMPTZ NOT NULL DEFAULT now()
```

#### Notification

```sql
notification_id   UUID PRIMARY KEY
user_id           UUID        NOT NULL REFERENCES user(user_id)
title             VARCHAR(255) NOT NULL
message           TEXT         NOT NULL
notification_type VARCHAR(50)  NOT NULL
is_read           BOOLEAN      NOT NULL DEFAULT FALSE
is_delivered      BOOLEAN      NOT NULL DEFAULT FALSE
created_at        TIMESTAMPTZ  NOT NULL DEFAULT now()
delivered_at      TIMESTAMPTZ
```

#### NotificationOutbox *(Transactional)*

```sql
outbox_id         UUID PRIMARY KEY
user_id           UUID        NOT NULL REFERENCES user(user_id)
title             VARCHAR(255) NOT NULL
message           TEXT         NOT NULL
notification_type VARCHAR(50)  NOT NULL
schedule_version_id UUID REFERENCES schedule_version(schedule_version_id)
created_at        TIMESTAMPTZ NOT NULL DEFAULT now()
processed_at      TIMESTAMPTZ
retry_count       INTEGER NOT NULL DEFAULT 0
status            VARCHAR(50) NOT NULL DEFAULT 'pending'
                  -- pending|processing|delivered|failed
```

---

## 8. Engine 3 — Operating Systems (20%)

### Purpose

Manage independent scheduling jobs, multiprocessing, inter-process communication, synchronization, job priorities and failure recovery.

### Finalized OS Components

1. Multiprocessing
2. Priority-Based Scheduling
3. Message Queues (IPC)
4. Mutex
5. Semaphores
6. Deadlock Prevention
7. Worker Pool
8. Failure Detection and Recovery
9. Aging

---

### 8.1 Multiprocessing

Use actual **Python `multiprocessing`** — not threading.

**Logical processes:**

| Process | Responsibilities |
|---|---|
| **Scheduling Coordinator** | Receive jobs, create job records, assign priorities, dispatch, track status |
| **Conflict Detector** | Identify conflicting activities, check affected resources, report conflicts |
| **Scheduling Worker** | Execute DSA algorithms, generate candidate schedules, validate results |
| **Result Handler** | Process worker results, update job status, store valid results |
| **Notification Worker** | Retrieve committed notification events, deliver notifications, handle retries |

> These are **logical responsibilities** implemented using a configurable worker pool.

---

### 8.2 Priority-Based Scheduling

**Job priority table:**

| Job Type | Priority |
|---|---|
| Emergency resource recovery | Highest (0) |
| Urgent conflict resolution | High (1) |
| Routine timetable update | Medium (2) |
| Initial timetable generation | Normal (3) |
| Background analytics | Low (4) |

> This is **application-level job scheduling**, not a replacement for the OS CPU scheduler.

---

### 8.3 Message Queues — IPC

```
Scheduling Coordinator
        │
        ▼
     Job Queue
        │
        ▼
  Scheduling Worker
        │
        ▼
    Result Queue
        │
        ▼
    Result Handler
```

**Job message fields:**

```python
{
    "job_id": UUID,
    "job_type": str,
    "priority": int,
    "schedule_version_id": UUID,
    "payload_reference": dict,
    "creation_time": datetime,
    "retry_count": int
}
```

> Do **not** share unmanaged database connections between processes. Workers use their own managed database sessions.

---

### 8.4 Mutex

Protect shared in-memory critical sections from simultaneous access.

---

### 8.5 Semaphores

Control access to limited processing resources or coordinate processes.

---

### 8.6 Deadlock Prevention

Strategies:
- Consistent lock ordering
- Minimal critical sections
- Lock timeouts
- Avoid unnecessary nested locks

> Create a **controlled test** demonstrating a deadlock scenario, then demonstrate the prevention strategy.  
> Do **not** introduce artificial locks into normal production workflows.

---

### 8.7 Worker Pool

Maintain reusable processes.

| Feature | Description |
|---|---|
| Worker initialization | Start N workers on application startup |
| Job assignment | Assign jobs from queue to available workers |
| Health monitoring | Detect crashed or hung workers |
| Graceful shutdown | Allow current jobs to complete before stopping |
| Job timeout | Detect and cancel jobs running too long |
| Failure detection | Identify and handle worker crashes |

---

### 8.8 Aging

Gradually increase the **effective priority** of jobs that have been waiting a long time.

**Prevents starvation** of lower-priority jobs.

```python
effective_priority = base_priority - (time_waiting / aging_interval)
```

---

### 8.9 Failure Recovery

```
Worker crashes
      │
      ▼
1. Detect failure (health monitor)
2. Mark job as INTERRUPTED
3. Determine whether retry is safe
4. Requeue eligible jobs
5. Enforce retry limits (max_retries)
6. Log failure information
7. Validate results before saving
```

> Use idempotency protection to avoid **duplicate side effects** on retry.

### 8.10 Job States

```
PENDING
    │
    ▼
QUEUED
    │
    ▼
RUNNING
    │
    ├──► COMPLETED
    ├──► FAILED ──► RETRYING ──► QUEUED (loop up to max_retries)
    └──► CANCELLED
```

---

## 9. Engine 4 — Web Technologies and Networks (20%)

### Finalized Technology Stack

| Component | Technology |
|---|---|
| Frontend | React |
| Language | TypeScript |
| Styling | Tailwind CSS |
| Backend | FastAPI |
| API communication | REST |
| Real-time updates | WebSockets |
| Authentication | JWT |
| Authorization | RBAC |
| Deployment | Docker + Docker Compose |

---

### 9.1 Frontend Pages

| # | Page | Primary Role |
|---|---|---|
| 1 | Login | All |
| 2 | Admin Dashboard | Admin |
| 3 | Department Dashboard | Coordinator |
| 4 | Faculty Dashboard | Faculty |
| 5 | Student Dashboard | Student |
| 6 | Resource Management | Admin, Coordinator |
| 7 | Timetable Management | Admin, Coordinator |
| 8 | Schedule Generation | Admin |
| 9 | Conflict Center | Admin, Coordinator |
| 10 | Simulation Center | Admin |
| 11 | Alternative Comparison | Admin |
| 12 | Approval Center | Admin |
| 13 | Audit Logs | Admin |
| 14 | Notifications | All |

---

### 9.2 REST API Endpoints

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

> Long-running scheduling requests must return a **job or simulation identifier** immediately.

---

### 9.3 WebSocket Events

Use WebSockets for **real-time only**. Use REST for ordinary CRUD operations.

| Event | Description |
|---|---|
| `JOB_PROGRESS` | Job progress update (0–100%) |
| `JOB_COMPLETED` | Job finished successfully |
| `JOB_FAILED` | Job failed with error |
| `SIMULATION_READY` | Simulation alternatives available |
| `SCHEDULE_UPDATED` | New active schedule version |
| `NOTIFICATION` | New notification for user |
| `WORKER_STATUS` | Worker pool status (admin only) |
| `CONFLICT_DETECTED` | New conflict found |

---

### 9.4 JWT and RBAC

- Secure password hashing: Argon2 or bcrypt
- JWT tokens for session management
- Role-based access control enforced **in the backend**
- WebSocket connections also require valid JWT

---

## 10. Most Important End-to-End Workflow

### Scenario: Lab 3 becomes unavailable for two days

This scenario must demonstrate all four engines working together.

#### Initial State
- Lab 3 is assigned to multiple practical classes
- A valid timetable exists with an active schedule version

| Step | Actor | Action | Engine |
|---|---|---|---|
| 1 | Admin | Opens Resource Management, selects Lab 3, enters start/end date and reason, submits | Web |
| 2 | FastAPI | Validates request, stores unavailability record | DBMS |
| 3 | Conflict Detector | Identifies all Lab 3 assignments during the affected period | DSA + DBMS |
| 4 | OS Coordinator | Creates high-priority recovery job, adds to priority queue | OS |
| 5 | Scheduling Worker | Runs BFS + CSP + Backtracking + Forward Checking | DSA |
| 6 | DBMS | Stores candidate schedules as simulation; active timetable unchanged | DBMS |
| 7 | Frontend | Displays affected activities, alternatives, metrics | Web |
| 8 | Admin | Selects an alternative | Web |
| 9 | Backend | Verifies authorization (admin role) | Web + DBMS |
| 10 | Backend | Validates that active version matches simulation's base version | DBMS |
| 11 | Backend | BEGIN TRANSACTION → validate → apply → new version → audit → outbox → COMMIT | DBMS |
| 12 | Notification Worker | Retrieves outbox entries, delivers notifications to affected users | OS + DBMS |
| 13 | WebSocket | Broadcasts `SCHEDULE_UPDATED` event | Web |
| 14 | Frontend | Displays new active timetable | Web |

> **This complete scenario must be implemented and tested.**

---

## 11. Two Primary Scheduling Workflows

### Workflow A: Initial Timetable Generation

```
Register Resources
       │
       ▼
Configure Constraints
       │
       ▼
Create Scheduling Activities
       │
       ▼
Build Conflict Graph
       │
       ▼
Conflict Graph & Invariant Verification
       │
       ▼
CSP Domain Construction
       │
       ▼
Backtracking + Forward Checking
       │
       ▼
Hard Constraint Validation
       │
       ▼
Soft Constraint Optimization
       │
       ▼
Generate Timetable
       │
       ▼
Administrator Review
       │
       ▼
Publish Timetable
```

### Workflow B: Incremental Schedule Recovery

```
Existing Active Timetable
         │
         ▼
Resource Becomes Unavailable
         │
         ▼
Identify Affected Assignments
         │
         ▼
Create Recovery Job (High Priority)
         │
         ▼
Analyze Affected Activities (DSA)
         │
         ▼
Generate Alternatives
         │
         ▼
Validate Complete Timetable
         │
         ▼
Rank Valid Alternatives
         │
         ▼
What-If Simulation (no active change)
         │
         ▼
Administrator Approval
         │
         ▼
Version Validation
         │
         ▼
Transactional Commit
         │
         ▼
Publish Updated Timetable
```

---

## 12. Scheduling Mathematical Model

Each scheduling activity is a variable:

```
Activity ID:       ACT-101
Course:            Deep Learning
Faculty:           Faculty 1
Batch:             BCI-A
Duration:          1 hour
Required room:     Laboratory
Required equipment: GPU-enabled systems
```

**A feasible assignment includes:**
- Activity
- Date or day
- Start time
- End time
- Room
- Required equipment

### Hard Constraint Check (Time Interval Overlap)

```python
def overlaps(slot_a, slot_b):
    # Must check actual clock times, not slot IDs
    return slot_a.start_time < slot_b.end_time and \
           slot_b.start_time < slot_a.end_time
```

**Example:** 9:00–10:00 and 9:30–10:30 → **overlap** even with different slot IDs.

### Soft Constraint Objective

$$\text{Minimize: } C = w_1 D + w_2 P + w_3 G + w_4 U$$

Hard constraints validated **independently** — never compromised.

---

## 13. Functional Requirements

| ID | Requirement | Description |
|---|---|---|
| FR-01 | Resource Management | Manage rooms, labs, faculty, courses, batches and equipment |
| FR-02 | Schedule Generation | Generate valid academic timetables |
| FR-03 | Conflict Detection | Detect room, faculty, batch and resource conflicts |
| FR-04 | Alternative Generation | Find alternative resources and time slots |
| FR-05 | What-If Simulation | Simulate changes without modifying the active timetable |
| FR-06 | Schedule Approval and Update | Approve and commit proposed changes |
| FR-07 | Database Management | Maintain persistent data and integrity |
| FR-08 | Notifications and Audit Logs | Notify users and track modifications |

---

## 14. Non-Functional Requirements

| ID | Requirement | Description |
|---|---|---|
| NFR-01 | Performance | Scheduling and conflict detection within defined time limits |
| NFR-02 | Scalability | Support growing resources and scheduling activities |
| NFR-03 | Reliability | Recover from worker failures |
| NFR-04 | Security | Authentication and authorization |
| NFR-05 | Data Integrity | Maintain transactional consistency |
| NFR-06 | Usability | Provide a clear dashboard |
| NFR-07 | Maintainability | Use modular architecture |
| NFR-08 | Responsiveness | Provide timely progress and notifications |

---

## 15. Use Cases

| ID | Use Case | Actor |
|---|---|---|
| UC-01 | Manage Resources | Administrator |
| UC-02 | Generate Schedule | Administrator |
| UC-03 | Detect Conflicts | Administrator / Coordinator |
| UC-04 | Find Alternatives | Coordinator |
| UC-05 | Simulate Changes | Administrator |
| UC-06 | Approve and Commit Changes | Administrator |
| UC-07 | View Timetable | Faculty / Student |
| UC-08 | Receive Notifications | Faculty / Student |
| UC-09 | View Audit Logs | Administrator |
| UC-10 | Manage Constraints | Administrator |

---

## 16. User Stories

| ID | Story |
|---|---|
| US-01 | As an administrator, I want to manage university resources so scheduling information remains updated. |
| US-02 | As an administrator, I want to generate timetables automatically. |
| US-03 | As a coordinator, I want to detect scheduling conflicts. |
| US-04 | As a coordinator, I want alternative room and time-slot suggestions. |
| US-05 | As an administrator, I want to simulate changes without modifying the active timetable. |
| US-06 | As an administrator, I want to compare alternatives to minimize disruption. |
| US-07 | As an administrator, I want to approve and commit changes safely. |
| US-08 | As a faculty member, I want to receive timetable notifications. |
| US-09 | As a student, I want to view the latest published timetable. |
| US-10 | As an administrator, I want to review audit history. |

---

## 17. Database Entity Relationships

```
DEPARTMENT
    ├── FACULTY
    ├── COURSE
    └── STUDENT BATCH

COURSE
    └── COURSE OFFERING
            ├── FACULTY
            └── STUDENT BATCH

ROOM
    ├── LABORATORY
    └── ROOM EQUIPMENT ── EQUIPMENT

COURSE OFFERING
    └── ASSIGNMENT
            ├── ROOM
            ├── TIME SLOT
            └── SCHEDULE VERSION

SCHEDULE VERSION
    ├── SIMULATION
    └── ASSIGNMENT

SIMULATION
    └── SIMULATION ASSIGNMENT

USER
    ├── AUDIT LOG
    └── NOTIFICATION

NOTIFICATION OUTBOX ── USER
```

---

## 18. Key Performance Indicators

> These are proposed acceptance targets. Must be **measured**, not assumed.

### Functional KPIs

| KPI | Target |
|---|---:|
| Valid resource operations | ≥ 98% |
| Generated schedules satisfying hard constraints | 100% |
| Correct detection of known conflicts | ≥ 95% |
| Feasible recovery scenarios with valid alternatives | ≥ 90% |
| Active timetable modifications before approval | 0 |
| Successful valid schedule commits | ≥ 99% |
| DB operations preserving integrity | 100% |
| Audit coverage of important modifications | 100% |

### Non-Functional KPIs

| KPI | Target |
|---|---|
| Generate timetable for 100 activities | ≤ 10 seconds |
| Process 500 activities | No hard-constraint violations |
| Eligible failed worker jobs recovered | ≥ 95% |
| Unauthorized access attempts rejected | 100% |
| Failed transactions leave DB consistent | 100% |
| Core-module test coverage | ≥ 70% |
| Read-only API P95 response time | ≤ 500 ms |
| Resource utilization | ≥ 75% |
| Schedule disruption rate (recovery) | ≤ 10% |
| Recovery time for predefined test scenario | ≤ 5 seconds |

---

## 19. Security Requirements

| Requirement | Implementation |
|---|---|
| Password storage | Argon2 or bcrypt hashing |
| Session management | JWT tokens |
| Authorization | Backend RBAC |
| Input validation | Pydantic models |
| SQL injection protection | SQLAlchemy ORM (parameterized) |
| Environment secrets | `.env` files, never committed |
| CORS | Configured explicitly |
| Rate limiting | Sensitive endpoints (login, etc.) |
| DB access | Least-privilege database user |
| WebSocket auth | JWT token required |
| Audit trail | All important actions logged |

---

## 20. Testing Strategy

### DSA Testing

**Functionality:**
- Graph construction and edge detection
- BFS impact analysis and component isolation
- CSP domain construction
- Backtracking with state restoration
- Forward Checking domain pruning
- Hard constraint validation
- Soft constraint scoring
- Incremental rescheduling
- Alternative ranking

**Edge cases:**
- No available rooms
- No available time slots
- Insufficient room capacity
- Faculty unavailable
- Missing equipment
- No feasible solution

### DBMS Testing

- Foreign key and unique constraints
- Room overlap prevention (GiST)
- Faculty overlap prevention
- Transactions and rollback
- Schedule version validation
- Concurrent approval conflict
- Audit log creation

### OS Testing

- Multiple workers
- Queue ordering by priority
- Aging behavior
- IPC message delivery
- Mutex protection
- Semaphore coordination
- Worker crash detection
- Retry limit enforcement
- Graceful shutdown
- Deadlock prevention

### Web Testing

- Authentication (login, JWT)
- RBAC enforcement
- Resource CRUD
- Schedule generation flow
- Simulation creation and completion
- Approval and commit
- Notification delivery
- WebSocket events

### End-to-End Testing

The **Lab 3 unavailability scenario** must work completely — from the first administrator action through the final notification and dashboard update.

---

## 21. Frontend Design Requirements

**Design direction:** Modern university administration platform

| Aspect | Requirement |
|---|---|
| Typography | Professional, clean, consistent |
| Layout | Responsive (desktop, tablet, mobile) |
| Navigation | Professional sidebar |
| Visual hierarchy | Clear section and content priority |
| Spacing | Consistent throughout |
| Contrast | Accessible (WCAG-compliant) |
| Analytics | Charts and dashboards |
| Feedback | Loading indicators, error messages |
| Confirmation | Dialogs for destructive or approval actions |
| Notifications | Toast notifications |

**Visual distinctions for assignment states:**

| State | Visual Treatment |
|---|---|
| Available resource | Default / green indicator |
| Unavailable resource | Grey / strikethrough |
| Conflicting assignment | Red highlight |
| Proposed assignment | Yellow / amber |
| Approved assignment | Blue / success |
| Active schedule version | Prominent badge |

---

## 22. Project Folder Structure

```
AURO/
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── layouts/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── context/
│   │   ├── types/
│   │   └── utils/
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
│   │   │
│   │   ├── dsa/
│   │   │   ├── graph.py
│   │   │   ├── bfs.py
│   │   │   ├── csp.py
│   │   │   ├── backtracking.py
│   │   │   ├── forward_checking.py
│   │   │   ├── priority_queue.py
│   │   │   ├── hash_maps.py
│   │   │   ├── constraint_validator.py
│   │   │   ├── optimizer.py
│   │   │   ├── conflict_detector.py
│   │   │   └── incremental_scheduler.py
│   │   │
│   │   ├── os_engine/
│   │   │   ├── coordinator.py
│   │   │   ├── worker.py
│   │   │   ├── worker_pool.py
│   │   │   ├── job_queue.py
│   │   │   ├── ipc.py
│   │   │   ├── synchronization.py
│   │   │   ├── recovery.py
│   │   │   └── notification_worker.py
│   │   │
│   │   ├── websocket/
│   │   │   ├── manager.py
│   │   │   └── events.py
│   │   │
│   │   └── tests/
│   │       ├── unit/
│   │       ├── integration/
│   │       └── performance/
│   │
│   ├── requirements.txt
│   └── Dockerfile
│
├── docs/
│   ├── INDEX.md
│   ├── COMPLETE_SPECIFICATION.md   ← this file
│   ├── MASTER_PROMPT.md
│   ├── architecture.md
│   ├── database_schema.md
│   ├── algorithms.md
│   ├── api_documentation.md
│   └── testing.md
│
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## 23. Development Phases

| Phase | Activity |
|---|---|
| **1** | Requirements and architecture |
| **2** | Database schema and migrations |
| **3** | Resource management |
| **4** | DSA algorithm implementation |
| **5** | Initial timetable generation |
| **6** | OS multiprocessing and workers |
| **7** | Conflict detection and recovery |
| **8** | Simulation and alternatives |
| **9** | Approval, versioning and transactions |
| **10** | Frontend dashboards |
| **11** | WebSockets and notifications |
| **12** | Testing and deployment |

> After each phase: list files created/modified, explain the implementation, run tests, identify and fix failures, report what is complete and what remains.

---

## 24. Sample Data

### Seed Script Requirements

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
| Faculty availability | All faculty × all slots |
| Course offerings | Enough for realistic conflicts |

### Demo Accounts (Development Only)

| Role | Email | Password |
|---|---|---|
| Administrator | admin@auro.edu | (set in seed script) |
| Coordinator | coordinator@auro.edu | (set in seed script) |
| Faculty | faculty@auro.edu | (set in seed script) |
| Student | student@auro.edu | (set in seed script) |

> Document credentials in **development documentation only**. Never commit production credentials.

### Demonstration Scenario

The seed data must enable a complete demonstration of:

1. Initial timetable generation
2. Conflict detection
3. Lab 3 unavailability (2 days)
4. Alternative schedule generation
5. What-if simulation (no active timetable change)
6. Administrator comparison and approval
7. Transactional commit
8. Notifications to affected faculty and students

---

## 25. Documentation

Create the following documents:

| Document | Contents |
|---|---|
| `README.md` | Overview, installation, startup, demo instructions |
| `docs/architecture.md` | System architecture, layer descriptions, diagrams |
| `docs/database_schema.md` | ER diagram, table definitions, indexing strategy |
| `docs/algorithms.md` | DSA algorithm explanations and pseudocode |
| `docs/api_documentation.md` | REST endpoints, WebSocket events, schemas |
| `docs/testing.md` | Test plan, test cases, coverage targets |

**Each document must explain how its engine contributes to AURO.**

---

## 26. Strict Implementation Rules

| # | Rule |
|---|---|
| 1 | Do not create a frontend-only prototype |
| 2 | Do not hardcode timetable results |
| 3 | Do not substitute random assignment for the selected algorithms |
| 4 | Do not violate hard constraints |
| 5 | Do not modify the active timetable during simulation |
| 6 | Do not approve schedules without authorization |
| 7 | Do not commit stale simulations |
| 8 | Do not claim multiprocessing when execution is sequential |
| 9 | Do not claim IPC without actual process communication |
| 10 | Do not claim synchronization without implementing it |
| 11 | Do not claim transactions without actual database transactions |
| 12 | Do not send notifications before successful commit |
| 13 | Do not leave frontend buttons disconnected |
| 14 | Do not silently ignore errors |
| 15 | Do not expose sensitive data in API responses |
| 16 | Do not store plaintext passwords |
| 17 | Do not replace PostgreSQL with MongoDB or SQLite |
| 18 | Do not replace CSP and Backtracking with generic greedy scheduling |
| 19 | Do not put the whole application in one file |
| 20 | Do not claim performance targets without benchmarks |
| 21 | Do not mark features complete before tests pass |

> If a feature cannot be implemented correctly, **explain the limitation** and propose a technically sound alternative.

---

## 27. Final Acceptance Checklist

### Authentication & Authorization
- [ ] Authentication (JWT login)
- [ ] Role-based authorization (backend enforced)

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
- [ ] BFS impact traversal
- [ ] CSP modeling
- [ ] Backtracking
- [ ] Forward Checking
- [ ] Priority Queue
- [ ] Hash Maps
- [ ] Hard constraint validation
- [ ] Soft constraint optimization

### Scheduling Intelligence
- [ ] Conflict detection
- [ ] Incremental schedule recovery
- [ ] Alternative schedule generation
- [ ] What-if simulation
- [ ] Active timetable preservation during simulation
- [ ] Alternative comparison

### Database Safety
- [ ] Administrator approval flow
- [ ] Schedule versioning
- [ ] Transactional schedule commit
- [ ] Transaction rollback
- [ ] Concurrency control (optimistic + pessimistic)

### OS Engine
- [ ] Multiprocessing (actual `multiprocessing`)
- [ ] IPC message queues
- [ ] Priority-based job scheduling
- [ ] Aging
- [ ] Worker Pool
- [ ] Mutex and Semaphores
- [ ] Deadlock prevention
- [ ] Worker failure recovery

### Web & Communication
- [ ] REST APIs
- [ ] WebSockets
- [ ] Notifications
- [ ] Audit logs
- [ ] Responsive dashboards

### Quality & Deployment
- [ ] Automated tests (≥70% coverage)
- [ ] Docker deployment
- [ ] Complete documentation
- [ ] End-to-end Lab 3 unavailability demonstration

---

## 28. Final Project Definition

**AURO (Autonomous University Resource Orchestrator)** is an intelligent university resource management and scheduling platform that integrates:

| Engine | Technology | Contribution |
|---|---|---|
| DSA | Conflict Graph, BFS, Priority Queue, Hash Maps, CSP, Backtracking, Forward Checking | 35% |
| DBMS | PostgreSQL, Normalization, Indexing, ACID, Concurrency, Versioning | 25% |
| OS | Multiprocessing, Priority Scheduling, IPC, Synchronization, Worker Pool | 20% |
| Web | React, TypeScript, Tailwind, FastAPI, REST, WebSockets, JWT, RBAC | 20% |

**The defining capability of AURO** is its ability to **generate, simulate, compare and safely apply alternative university schedules while minimizing disruption to existing academic operations.**

---

## 29. Execution Instructions

Before implementing anything:

1. Read and understand this complete specification
2. Inspect the available development environment
3. Identify existing project files
4. Produce a complete technical architecture
5. Produce a detailed database ER design
6. Produce the DSA algorithm design
7. Produce the OS process communication design
8. Produce the API design
9. Produce a phase-by-phase implementation roadmap

Then begin Phase 1.

**After each phase:**
- List the files created or modified
- Explain the implementation
- Run relevant tests
- Identify failures
- Fix failures
- Report what is complete and what remains

> Do not skip phases.  
> Do not stop after generating a frontend.  
> Do not generate fake data responses to imitate working features.  
> Do not claim functionality is complete unless it has been implemented and tested.
>
> **The final deliverable must be a fully integrated, functional AURO application that demonstrates all four engineering domains through actual working software.**

---

*This document is the **single source of truth** for AURO development. All design decisions, algorithm selections, technology choices and architectural patterns defined here are finalized. Do not arbitrarily replace finalized algorithms, technologies or engineering concepts.*

