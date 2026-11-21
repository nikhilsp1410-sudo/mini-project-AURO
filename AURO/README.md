# AURO — Autonomous University Resource Orchestrator

AURO is an intelligent academic resource management and timetable orchestration platform designed to automate university scheduling, detect resource conflicts, simulate what-if disruptions, and execute minimal-displacement schedule recovery.

---

## The Four Engineering Domains

| Domain                                       | Contribution | Core Focus                                                                                                                             |
| -------------------------------------------- | :-----------: | -------------------------------------------------------------------------------------------------------------------------------------- |
| **Data Structures & Algorithms (DSA)** | **35%** | Conflict Graph Theory, BFS Impact Traversal, Priority Queue (Min-Heap), Hash Maps, CSP Modeling, Backtracking with Forward Checking, Minimum-Disruption Recovery. |
| **Database Management Systems (DBMS)** | **25%** | PostgreSQL Relational Schema (19 tables), ACID Transactions, Schedule Versioning, Optimistic Locking, Transactional Outbox.            |
| **Operating Systems (OS)**             | **20%** | Multiprocessing Worker Pool, Priority-based Job Scheduling with Dynamic Aging, IPC Message Queues, Havender Deadlock Prevention.       |
| **Web Technologies & Networks**        | **20%** | React 18, TypeScript, Tailwind CSS 2D Matrix Grid, FastAPI REST APIs, Real-time WebSockets, JWT Authentication, Backend RBAC.          |

---

## Authoritative Documentation

All architectural specifications, database designs, algorithm pseudocode, and API schemas are documented in the `docs/` directory:

- [**AURO_Revised_DSA_Engine.md**](docs/AURO_Revised_DSA_Engine.md) — Authoritative specification for pre-built timetable validation, conflict detection & recovery.
- [**COMPLETE_SPECIFICATION.md**](docs/COMPLETE_SPECIFICATION.md) — The authoritative single source of truth for all requirements and workflows.
- [**KLE_TECH_REFERENCE_TIMETABLE_SPECIFICATION.md**](docs/KLE_TECH_REFERENCE_TIMETABLE_SPECIFICATION.md) — Ground-truth academic reference benchmark (CSE 5th Sem Divisions A–D, exact slots, co-faculty teams, 2/3/4-hr lab blocks).
- [**ENGINEERING_CAMPUS_SPECIFICATION.md**](docs/ENGINEERING_CAMPUS_SPECIFICATION.md) — 4-department multi-building campus modeling.
- [**CONFLICT_MODELS_AND_SCENARIOS.md**](docs/CONFLICT_MODELS_AND_SCENARIOS.md) — 7 invariant conflict classes and 5 concrete failure scenarios.
- [**architecture.md**](docs/architecture.md) — 5-layer system architecture, process communications, and security stack.
- [**database_schema.md**](docs/database_schema.md) — Complete ER design, table DDLs, index strategies, and concurrency controls.
- [**algorithms.md**](docs/algorithms.md) — Conflict graph, BFS impact analysis, Priority Queue, Hash Maps, CSP, Backtracking & Forward Checking.
- [**api_documentation.md**](docs/api_documentation.md) — Complete REST endpoints and WebSocket event specifications.
- [**testing.md**](docs/testing.md) — Multi-tier test suite plan and the 15-step Lab 3 Outage scenario test.
- [**INDEX.md**](docs/INDEX.md) — Project index and documentation registry.

---

## Localhost Quickstart (No Docker Required)

### 1. Start FastAPI Backend (Port 8000)

```powershell
cd backend
python -m uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload
```

*Swagger OpenAPI Documentation:* http://127.0.0.1:8000/docs

### 2. Start React Frontend (Port 5173)

```powershell
cd frontend
npm run dev
```

*Frontend URL:* http://localhost:5173

---

## Authentic Seeded Accounts (1-Click Switcher Available on Login Page)

| Role                          | Email                    | Password       | Persona & Department Affiliation                                                 |
| ----------------------------- | ------------------------ | -------------- | -------------------------------------------------------------------------------- |
| **Executive Admin**     | `admin@auro.edu`       | `admin123`   | **Dr. Praveen A. Ghorpade** (Dean Academics & Campus Administrator)        |
| **Department Admin**    | `hod.cse@auro.edu`     | `admin123`   | **Dr. Rajashri Khanai** (Professor & Head of Department, CSE)              |
| **Coordinator**         | `coordinator@auro.edu` | `coord123`   | **Dr. Priyanka Gavade** (Timetable Officer & Coordinator, CSE)             |
| **Teaching Faculty**    | `salma@auro.edu`       | `faculty123` | **Dr. Salma Shahapur** (Associate Professor, Deep Learning & DL Lab)       |
| **Teaching Faculty**    | `amey@auro.edu`        | `faculty123` | **Prof. Amey Muchandi** (Assistant Professor, Embedded Systems & MCES Lab) |
| **Student (Section A)** | `student.a@auro.edu`   | `student123` | **Rohan Deshmukh** (V Sem B.E. CSE - Div A, Hall: Step Hall)               |
| **Student (Section D)** | `student.d@auro.edu`   | `student123` | **Pooja Hegde** (V Sem B.E. CSE - Div D AI/GPU Track, Hall: AI Room)       |
