# AURO — 4-Department Engineering College Campus Specification
## Multi-Building, Multi-Division Institutional Blueprint

> **Document Type:** Campus Architecture & Domain Modeling Specification  
> **Status:** Active Blueprint  
> **Target Environment:** 4-Department Engineering College (4 Academic Years, 5–6 Divisions per Year)  
> **Last Updated:** 2026-11-11

---

## 1. Executive Summary & Campus Scale

AURO is configured specifically for a **multi-building Engineering College** comprising **4 distinct academic engineering departments**. Each department operates in its own dedicated campus building, offering a 4-year undergraduate curriculum with 5 to 6 divisions (cohort sections) per year.

### Campus Metrics at a Glance

| Metric | Dimension | Institutional Total |
|---|---|---|
| **Departments** | 4 Engineering Disciplines | **4 Departments** |
| **Buildings / Blocks** | 1 dedicated building per department | **4 Campus Blocks** |
| **Academic Years** | 4 Years per department (Semesters 1 through 8) | **4 Years (8 Semesters)** |
| **Divisions per Year** | 5 to 6 Divisions per year per department | **20 to 24 Divisions / Dept** |
| **Total Student Divisions** | 4 departments × ~22 divisions average | **~88 Student Cohorts** |
| **Average Division Size** | ~60 students per division | **~5,200 Active Students** |
| **Faculty Pool** | ~30–40 professors/lecturers per department | **~140–160 Faculty Members** |
| **Weekly Sessions Scheduled** | ~25–30 contact hours per division | **~2,200–2,600 Class Sessions/Week** |

---

## 2. Campus Infrastructure & Building Topology

Every department is housed in an independent physical building on campus. This establishes a **spatial constraint**: activities should strictly take place within the department's assigned building to prevent student/faculty transit delays across campus, unless specialized inter-departmental shared facilities are required.

```
┌─────────────────────────┐         ┌─────────────────────────┐
│       BLOCK A (CSE)     │         │       BLOCK B (ECE)     │
│ Computer Science & Eng. │         │ Electronics & Comm. Eng.│
│ ├── 12 Classrooms       │         │ ├── 12 Classrooms       │
│ ├── 6 Computer Labs     │         │ ├── 6 Hardware/VLSI Labs│
│ └── Faculty Wing A      │         │ └── Faculty Wing B      │
└────────────┬────────────┘         └────────────┬────────────┘
             │                                   │
             └───────────── Campus Central ──────┘
             ┌───────────── Walkway/Plaza ───────┐
             │                                   │
┌────────────┴────────────┐         ┌────────────┴────────────┐
│      BLOCK C (MECH)     │         │      BLOCK D (CIVIL)    │
│ Mechanical Engineering  │         │   Civil Engineering     │
│ ├── 12 Classrooms       │         │ ├── 12 Classrooms       │
│ ├── 6 Workshop/Mech Labs│         │ ├── 6 Structural/Survey │
│ └── Faculty Wing C      │         │ └── Faculty Wing D      │
└─────────────────────────┘         └─────────────────────────┘
```

### Detailed Building Allocation

| Building | Department Code | Department Name | Classroom Facilities | Specialized Laboratories |
|---|:---:|---|---|---|
| **Block A** | `CSE` | Computer Science & Engineering | Rooms A101–A106 (L1)<br>Rooms A201–A206 (L2) | • Systems Programming Lab (Lab A-1)<br>• Data Science & AI Lab (Lab A-2)<br>• **Networks & OS Lab (Lab A-3)** *(Outage Scenario)*<br>• Cloud Computing Lab (Lab A-4)<br>• Web Technologies Lab (Lab A-5)<br>• High-Performance Computing Lab (Lab A-6) |
| **Block B** | `ECE` | Electronics & Communication Eng. | Rooms B101–B106 (L1)<br>Rooms B201–B206 (L2) | • Digital Signal Processing Lab<br>• VLSI & Microelectronics Lab<br>• Embedded Systems Lab<br>• Communication Systems Lab<br>• Analog Circuits Lab<br>• Microwave & Optical Lab |
| **Block C** | `MECH` | Mechanical Engineering | Rooms C101–C106 (L1)<br>Rooms C201–C206 (L2) | • CAD/CAM Simulation Center<br>• Thermal Engineering Lab<br>• Fluid Mechanics Lab<br>• Manufacturing & Machine Shop<br>• Materials Testing Lab<br>• Robotics & Mechatronics Lab |
| **Block D** | `CIVIL` | Civil Engineering | Rooms D101–D106 (L1)<br>Rooms D201–D206 (L2) | • Structural Engineering Lab<br>• Geotechnical & Soil Mechanics<br>• Environmental Engineering Lab<br>• Surveying & Geomatics Lab<br>• Concrete Technology Lab<br>• Hydraulics & Water Resources |

---

## 3. Academic Structure: Years, Semesters & Divisions

Each department runs a standard **4-Year (8-Semester)** engineering degree program. Every year is split into **5 to 6 parallel divisions** (Sections A through F).

### Nomenclature & Cohort Tagging

Every division has a unique institutional identifier formatted as:
`{DEPT}-{YEAR}{DIVISION}` (e.g. `CSE-3A`, `ECE-2C`, `MECH-4E`).

```
Department (e.g. CSE)
├── Year 1 (Freshman / Sem 1-2)  ──► Divisions: CSE-1A, CSE-1B, CSE-1C, CSE-1D, CSE-1E, CSE-1F
├── Year 2 (Sophomore / Sem 3-4) ──► Divisions: CSE-2A, CSE-2B, CSE-2C, CSE-2D, CSE-2E, CSE-2F
├── Year 3 (Junior / Sem 5-6)    ──► Divisions: CSE-3A, CSE-3B, CSE-3C, CSE-3D, CSE-3E, CSE-3F
└── Year 4 (Senior / Sem 7-8)    ──► Divisions: CSE-4A, CSE-4B, CSE-4C, CSE-4D, CSE-4E, CSE-4F
```

### Student Count per Division
- **Classroom Lectures:** Attended by the full division (~60 students). Classroom capacity must be $\ge 65$ seats.
- **Laboratory Practical Sessions:** Typically conducted in continuous 2-hour blocks. For high-demand equipment, divisions can either utilize full labs with 60 workstations or operate in dual batches.

---

## 4. Subject Taxonomy by Department (Representative Semester Sample)

Each department features a distinct curriculum divided into **Theory Lectures** (3–4 hrs/week) and **Practical Lab Sessions** (2–3 hrs/week).

### Department 1: Computer Science & Engineering (Block A)
- **Year 2 (Sem 3):** Data Structures & Algorithms (Theory + Lab), Digital Logic (Theory), Discrete Mathematics (Theory), Computer Organization (Theory).
- **Year 3 (Sem 5):** Operating Systems (Theory + Lab), Database Management Systems (Theory + Lab), Computer Networks (Theory + Lab), Theory of Computation (Theory).
- **Year 4 (Sem 7):** Distributed Systems (Theory), Machine Learning (Theory + Lab), Information Security (Theory), Capstone Project Phase 1.

### Department 2: Electronics & Communication Engineering (Block B)
- **Year 2 (Sem 3):** Electronic Devices & Circuits (Theory + Lab), Signals and Systems (Theory), Network Analysis (Theory), Digital System Design (Theory + Lab).
- **Year 3 (Sem 5):** Microprocessors & Microcontrollers (Theory + Lab), Electromagnetic Waves (Theory), Analog & Digital Communication (Theory + Lab), Control Systems (Theory).
- **Year 4 (Sem 7):** VLSI Design (Theory + Lab), Optical Communication (Theory), Wireless Networks (Theory), Project Work.

### Department 3: Mechanical Engineering (Block C)
- **Year 2 (Sem 3):** Thermodynamics (Theory), Mechanics of Solids (Theory + Lab), Material Science (Theory), Machine Drawing (CAD Lab).
- **Year 3 (Sem 5):** Heat & Mass Transfer (Theory + Lab), Design of Machine Elements (Theory), Fluid Power Control (Theory + Lab), Dynamics of Machinery (Theory).
- **Year 4 (Sem 7):** CAD/CAM/CIM (Theory + Lab), Refrigeration & Air Conditioning (Theory), Operations Research (Theory), Industrial Project.

### Department 4: Civil Engineering (Block D)
- **Year 2 (Sem 3):** Surveying (Theory + Field Lab), Mechanics of Materials (Theory + Lab), Fluid Mechanics (Theory + Lab), Building Construction (Theory).
- **Year 3 (Sem 5):** Structural Analysis (Theory), Geotechnical Engineering (Theory + Lab), Environmental Engineering (Theory + Lab), Transportation Eng. (Theory).
- **Year 4 (Sem 7):** Design of Steel Structures (Theory), Water Resources Engineering (Theory), Construction Management (Theory), Final Project.

---

## 5. Architectural Impact on the 4 AURO Engines

Scaling the institutional scope to 4 departments, 4 blocks, and ~88 divisions shapes how each engine operates:

### 5.1 DSA Engine Impact (35%)
1. **Building Locality Hard/Soft Constraint**:
   - *Hard:* Lab sessions must occur in laboratories equipped for that discipline (e.g. MECH Fluid lab cannot host CSE Operating Systems lab).
   - *Soft:* Students and faculty assigned to Department $D$ should have all lectures in Building $D$. Inter-building travel penalty $w_{transit} \times \text{distance}$ is added to the soft-constraint objective.
2. **Subproblem Decomposition & Independence**:
   - Because each department has its own building, classrooms, faculty, and student batches, routine timetables for the 4 departments are **partially orthogonal**.
   - Conflict graphs can be partitioned by department, with cross-edges only present for university-wide shared lecture halls (Auditorium, Seminar Hall) or shared foundation faculty (e.g. Mathematics/Physics in Year 1).
3. **Disruption Recovery Scoping (e.g. Lab A-3 Outage)**:
   - When **Lab A-3 (Block A - CSE)** undergoes maintenance:
     - Directly affected: Only CSE Year 3 divisions (e.g. `CSE-3A`, `CSE-3B`) using Lab A-3.
     - Search Space: The DSA engine first explores alternative labs **within Block A** (e.g. Lab A-1, Lab A-4) before expanding to cross-building labs.
     - Impact: ECE, MECH, and CIVIL schedules remain 100% unaffected.

### 5.2 DBMS Engine Impact (25%)
1. **Building & Spatial Attributes**:
   - `Room` model explicitly includes `building_code` (`BLOCK_A`, `BLOCK_B`, `BLOCK_C`, `BLOCK_D`) and `floor_number`.
2. **Division & Cohort Modeling**:
   - `StudentBatch` records store `department_id`, `academic_year` (1–4), `semester` (1–8), and `division_code` (`A`, `B`, `C`, `D`, `E`, `F`).
   - Composite index on `(department_id, semester, division_code)` for microsecond timetable retrieval.
3. **Department Coordinator Multi-Tenancy**:
   - `User` role `COORDINATOR` is bound to a specific `department_id`. A CSE Coordinator can report outages and approve changes **only for Block A**, while an Administrator holds campus-wide privileges.

### 5.3 OS Engine Impact (20%)
1. **Parallel Departmental Scheduling**:
   - The OS Coordinator can leverage Python `multiprocessing` to run **independent solver processes concurrently**:
     - Worker 1: Solves `CSE` schedule.
     - Worker 2: Solves `ECE` schedule.
     - Worker 3: Solves `MECH` schedule.
     - Worker 4: Solves `CIVIL` schedule.
   - This delivers true multi-core parallel speedup without lock contention.
2. **Prioritized Job Routing**:
   - A campus-wide emergency (e.g. power failure in Block C) queues an `EMERGENCY` priority recovery job that pre-empts routine background analytics.

### 5.4 Web & Networks Engine Impact (20%)
1. **Cascading Timetable Selector UI**:
   - Instant 4-tier filtering:
     `[Department Dropdown: CSE / ECE / MECH / CIVIL]`
     └── `[Academic Year: Year 1 / Year 2 / Year 3 / Year 4]`
         └── `[Division: Div A / Div B / Div C / Div D / Div E / Div F]`
2. **Multi-Perspective Views**:
   - **Student View:** Shows only their division's weekly schedule in their block.
   - **Faculty View:** Shows their teaching schedule across all assigned divisions and rooms.
   - **Coordinator View:** Building-wide matrix showing room occupancy across Block floors.
   - **Admin View:** Campus-wide heat map of room utilization across all 4 Blocks.

---

## 6. Demonstration Scenario: "Lab A-3 Maintenance Outage"

To prove the multi-department architecture works end-to-end, the standard demo scenario is localized to **Block A (Computer Science & Engineering)**:

1. **Active Campus State:** All 4 departments (Block A, B, C, D) have published, active Version 1.0 timetables.
2. **The Disruption Event:** The CSE Coordinator reports that **Lab A-3 (Networks & OS Lab)** is unavailable for 2 days due to server room maintenance.
3. **Targeted Detection:** The Conflict Detector isolates the 4 practical sessions belonging to `CSE-3A` and `CSE-3B` that were booked in Lab A-3.
4. **Isolated Incremental Reschedule:**
   - The DSA engine keeps Block B (ECE), Block C (MECH), and Block D (CIVIL) untouched.
   - It re-routes the sessions to **Lab A-1 (Systems Programming Lab)** during compatible available slots in Block A.
   - Zero changes to unrelated lecture classes.
5. **Approval & Commit:**
   - CSE Coordinator / University Administrator reviews the simulation comparison card.
   - Verifies 0 conflicts and 2 minimal lab reallocations.
   - Transactionally commits new Schedule Version 2.0.
   - Notification outbox sends push alerts to `CSE-3A` and `CSE-3B` students and their lab faculty.

---

## 7. Next Steps & Blueprint Checklist

- [ ] Add `building_code` and `floor` fields to the `Room` schema.
- [ ] Add `academic_year` (1–4) and `division` (`A`–`F`) fields to `StudentBatch`.
- [ ] Create seed generator scripts for the 4 departments, 4 blocks, and 24 divisions per department.
- [ ] Implement the cascading `Department > Year > Division` frontend selector component.

