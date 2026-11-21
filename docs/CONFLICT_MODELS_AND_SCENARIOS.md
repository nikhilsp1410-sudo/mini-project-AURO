# AURO — Conflict Models, Failure Scenarios & Recovery Specification
## Comprehensive Conflict Engineering for 4-Department Multi-Building Campus

> **Document Type:** Conflict Engineering & Problem Statement Specification  
> **Status:** Active Reference  
> **Target Environment:** 4-Department Engineering College (4 Buildings, 4 Years, ~88 Divisions)  
> **Last Updated:** 2026-11-11

---

## 1. Executive Purpose & Problem Statement Alignment

As established in Section 2 of the Master Specification, ordinary timetable generators only solve static scheduling once and break down when real-world academic disruptions occur. 

In a real engineering college operating across **4 departments**, **4 physical buildings**, **4 academic years**, and **5 to 6 divisions per year** (~88 student divisions, ~5,200 students), conflicts arise dynamically every week:

- Classrooms and laboratories suffer equipment or electrical breakdowns.
- Faculty members take emergency leaves or teach foundational courses across multiple department blocks.
- Central specialized facilities (like the central workshop or specialized computing clusters) face conflicting booking requests from different departments.
- Rooms are accidentally double-booked or assigned to divisions exceeding room capacity.

AURO’s core innovation is its **Conflict Detector** and **Incremental Schedule Recovery Engine**:
> **Detect the disruption, isolate the affected subproblem, generate minimal-displacement alternatives without disturbing unrelated academic operations, and safely commit approved changes.**

---

## 2. The 7 Conflict Classes (Mathematical Invariants)

Every scheduling assignment is defined as an assignment tuple:
$$\alpha = \langle act, room, slot, date\_or\_day \rangle$$
Where:
- $act$: Scheduling activity (Course, Faculty, Batch, Duration, Lab Requirement).
- $room$: Physical facility (Building, Floor, Capacity, Type, Equipment).
- $slot$: Continuous time interval on a given day $[T_{start}, T_{end})$.

A conflict occurs whenever an invariant $I_k$ is violated.

```
                    ┌───────────────────────────────┐
                    │     SCHEDULING ASSIGNMENT     │
                    │   Activity · Room · Slot      │
                    └───────────────┬───────────────┘
                                    │
                                    ▼
       ┌──────────────────── CONFLICT DETECTOR ──────────────────┐
       │                                                         │
       ├──► Class 1: Room Double-Booking (Spatial Overlap)        │
       ├──► Class 2: Faculty Contention (Cross-Dept / Cohort)   │
       ├──► Class 3: Division Double-Booking (Cohort Overlap)     │
       ├──► Class 4: Resource Unavailability (Facility Outage)   │
       ├──► Class 5: Capacity Deficiency (Overcrowding)          │
       ├──► Class 6: Equipment / Lab Mismatch (Missing Tech)     │
       └──► Class 7: Inter-Building Transit Collision            │
```

---

### Class 1: Room / Laboratory Double-Booking
- **Condition:** Two distinct activities $\alpha_1, \alpha_2$ occupy the exact same physical room during overlapping clock times on the same day.
- **Invariant:**
  $$\forall \alpha_1 \neq \alpha_2, \; room(\alpha_1) = room(\alpha_2) \implies slot(\alpha_1) \cap slot(\alpha_2) = \emptyset$$
- **Real-World Cause:** E.g., `CSE-2A` and `CSE-2B` both scheduled for lectures in Room A101 at 10:00 AM Monday.

---

### Class 2: Faculty Contention (Cross-Departmental Faculty Sharing)
- **Condition:** The same faculty member is assigned to teach two different activities during overlapping time intervals.
- **Invariant:**
  $$\forall \alpha_1 \neq \alpha_2, \; faculty(\alpha_1) = faculty(\alpha_2) \implies slot(\alpha_1) \cap slot(\alpha_2) = \emptyset$$
- **Real-World Cause:** Common in engineering colleges where:
  - Department of Mathematics faculty teach Calculus in Block A (`CSE-1A`) and also in Block B (`ECE-1B`).
  - Humanities / Management faculty teach Professional Ethics across CSE, MECH, and CIVIL.
  - If both departments schedule the shared professor at 11:00 AM on Tuesday, a **Faculty Contention Conflict** erupts.

---

### Class 3: Student Division Double-Booking
- **Condition:** The same student division (e.g. `ECE-3C`) is assigned to attend two different classes simultaneously.
- **Invariant:**
  $$\forall \alpha_1 \neq \alpha_2, \; batch(\alpha_1) = batch(\alpha_2) \implies slot(\alpha_1) \cap slot(\alpha_2) = \emptyset$$
- **Real-World Cause:** An elective lecture and a mandatory lab practical assigned to the same division during the same time window.

---

### Class 4: Resource Unavailability & Emergency Outage (The Core Scenario)
- **Condition:** An assignment utilizes a room, laboratory, or faculty member during a period marked by an active `ResourceUnavailability` entry.
- **Invariant:**
  $$\forall \alpha, \; (resource(\alpha), slot(\alpha)) \cap UnavailabilityPeriods(resource) = \emptyset$$
- **Real-World Cause:**
  - **Lab A-3 (Block A - CSE Networks Lab)** has an electrical fire or server room maintenance for 2 days.
  - **Prof. Sharma** has a sudden medical emergency and is unavailable Wednesday through Friday.
  - **Room C201 (Block C - MECH)** has an air conditioner/projector failure.

---

### Class 5: Room Capacity Deficiency
- **Condition:** The student count of the assigned division exceeds the physical seating capacity of the room.
- **Invariant:**
  $$\forall \alpha, \; Capacity(room(\alpha)) \ge StudentCount(batch(\alpha))$$
- **Real-World Cause:** A 65-student division (`MECH-2D`) assigned to a 40-seat tutorial room (C105). 25 students cannot sit.

---

### Class 6: Equipment & Laboratory Type Mismatch
- **Condition:** A course requires specific equipment or laboratory facilities, but the assigned room lacks them or is a regular classroom.
- **Invariant:**
  $$RequiresLab(course(\alpha)) \implies Type(room(\alpha)) = \text{'LAB'} \;\land\; Equipment(course) \subseteq Equipment(room)$$
- **Real-World Cause:**
  - Database Lab requiring Oracle/PostgreSQL workstations assigned to a standard lecture hall with only desks.
  - VLSI Design Lab requiring FPGA boards assigned to an analog circuits lab with only oscilloscopes.

---

### Class 7: Campus Transit Collision (Spatial Distance Violation)
- **Condition:** A student division or faculty member is scheduled in two different campus blocks in immediately consecutive time slots with 0 transit time.
- **Invariant:**
  $$\text{If } \alpha_1 \text{ ends at } T \text{ in Block } B_1 \text{ and } \alpha_2 \text{ begins at } T \text{ in Block } B_2 \text{ with } B_1 \neq B_2 \implies \text{Transit Conflict}$$
- **Real-World Cause:** A Year 1 division (`CIVIL-1B`) finishes a lecture in Block D at 11:00 AM and is scheduled for Workshop in Block C at 11:00 AM sharp without travel time.

---

## 3. Real-World Conflict Emergence in 4-Department Setup

Here is how conflicts naturally cross departmental boundaries in our 4-building campus:

```
                            ┌─────────────────────────────────┐
                            │    SHARED FOUNDATION FACULTY    │
                            │   Maths · Physics · Humanities  │
                            └───────┬──────────────┬──────────┘
                                    │              │
               ┌────────────────────┘              └────────────────────┐
               ▼                                                        ▼
    ┌──────────────────────┐                                 ┌──────────────────────┐
    │     BLOCK A (CSE)    │                                 │     BLOCK B (ECE)    │
    │  CSE-1A: Maths Class │ ◄─── CONFLICT: Shared Faculty ──► ECE-1B: Maths Class  │
    │  Slot: Mon 10:00 AM  │      Dr. K. Sharma              │  Slot: Mon 10:00 AM  │
    └──────────────────────┘                                 └──────────────────────┘

                                    ┌─────────────────────────────────┐
                                    │    SHARED CAMPUS FACILITIES     │
                                    │  Central Workshop (Block C)     │
                                    └───────┬──────────────┬──────────┘
                                            │              │
               ┌────────────────────────────┘              └────────────────────┐
               ▼                                                                ▼
    ┌──────────────────────┐                                         ┌──────────────────────┐
    │     BLOCK A (CSE)    │                                         │    BLOCK D (CIVIL)   │
    │  CSE-1C: Workshop    │ ◄─── CONFLICT: Double-Booked ──────────►│ CIVIL-1A: Workshop    │
    │  Slot: Tue 02:00 PM  │      Central Workshop (MECH Block C)    │ Slot: Tue 02:00 PM   │
    └──────────────────────┘                                         └──────────────────────┘
```

---

## 4. Five Standard Demonstration Scenarios

These 5 concrete test scenarios must be seeded and tested to prove AURO satisfies the problem statement:

---

### Scenario 1 (Master Problem Statement): 2-Day Maintenance Outage of Lab A-3
* **Location:** Block A (Computer Science & Engineering).
* **Facility:** Lab A-3 (Networks & Operating Systems Lab — 60 Linux workstations).
* **Initial State:** Active timetable Version 1.0. All assignments valid.
* **The Event:** Administrator submits `ResourceUnavailability`:
  - Resource: `Lab A-3`
  - Duration: Wednesday 08:00 to Thursday 18:00 (2 days).
  - Reason: *"Central UPS and Network Switch Upgrade"*.
* **Detected Conflicts:**
  - Activity 1: `CSE-3A` — Operating Systems Lab (Wed 10:00–12:00)
  - Activity 2: `CSE-3B` — Computer Networks Lab (Wed 14:00–16:00)
  - Activity 3: `CSE-3C` — Systems Programming Lab (Thu 09:00–11:00)
  - Activity 4: `CSE-3D` — Distributed Systems Lab (Thu 14:00–16:00)
* **Subproblem Isolation:**
  - Block B (ECE), Block C (MECH), and Block D (CIVIL) are **100% unaffected**.
  - All unaffected CSE lecture classrooms remain **fixed**.
* **Recovery Solution:**
  - DSA Incremental Scheduler relocates Activities 1 & 2 to **Lab A-1 (Systems Lab)**.
  - Relocates Activities 3 & 4 to **Lab A-5 (Cloud & Web Lab)** during open slots in Block A.
  - Alternative 1 score: Disruption = 4 changes, 0 hard violations.
* **Transactional Commit:**
  - Approved alternative committed into **Schedule Version 2.0**.
  - Notification outbox sends alerts only to `CSE-3A`, `CSE-3B`, `CSE-3C`, `CSE-3D` and their lab professors.

---

### Scenario 2: Cross-Department Shared Faculty Double-Booking
* **Location:** Block A (CSE) and Block B (ECE).
* **Actors:** Dr. K. Sharma (Senior Professor of Applied Mathematics).
* **The Conflict:**
  - Department Coordinator of CSE books Dr. Sharma for `CSE-1A` Engineering Calculus on Monday at 10:00 AM in Room A102.
  - Department Coordinator of ECE books Dr. Sharma for `ECE-1C` Linear Algebra on Monday at 10:00 AM in Room B101.
* **Detection:** Conflict Detector identifies overlap:
  - Invariant Violated: **Class 2 (Faculty Contention)**.
  - Severity: **CRITICAL (Hard Constraint)**.
* **Recovery Action:**
  - Relocates `ECE-1C` Calculus to Monday 14:00 PM in Room B101 when Dr. Sharma is free and Room B101 is unoccupied.

---

### Scenario 3: Central Workshop Contention
* **Location:** Block C (Mechanical Engineering Building — Ground Floor Central Workshop).
* **The Conflict:**
  - All 1st-year engineering students across all 4 departments have a mandatory "Engineering Workshop Practice" course (Carpentry, Welding, Fitting, Sheet Metal).
  - `CSE-1D` (60 students) is scheduled for Workshop on Tuesday 14:00–17:00.
  - `CIVIL-1B` (60 students) is scheduled for Workshop on Tuesday 15:00–18:00.
* **Detection:** Conflict Detector identifies a 2-hour overlap (15:00–17:00) exceeding workshop capacity:
  - Invariant Violated: **Class 1 (Room Double-Booking) & Class 5 (Capacity Overflow)**.
* **Recovery Action:**
  - Moves `CIVIL-1B` Workshop to Thursday afternoon (14:00–17:00).

---

### Scenario 4: Classroom Capacity Overflow
* **Location:** Block C (Mechanical Engineering Building).
* **The Conflict:**
  - Course: `MECH-301` Fluid Mechanics (Division `MECH-3A` with 65 students).
  - Assigned Room: Room C105 (Small Tutorial Room, capacity 40 seats).
* **Detection:**
  - Invariant Violated: **Class 5 (Capacity Deficiency: 65 > 40)**.
* **Recovery Action:**
  - Swaps Room C105 with Lecture Hall C201 (capacity 75 seats).

---

### Scenario 5: Sudden Faculty Medical Leave
* **Location:** Block B (Electronics & Communication Engineering).
* **The Event:**
  - Dr. Priya Nair (Associate Professor, ECE) falls ill on Thursday morning and logs emergency sick leave for Thursday and Friday.
* **Detection:**
  - Conflict Detector flags 3 assigned lectures for `ECE-2A`, `ECE-2B`, and `ECE-4C`.
* **Recovery Action:**
  - System identifies substitute faculty from ECE roster with matching subject expertise and available hours, or reschedules classes to open slots the following week.

---

## 5. Conflict Resolution Pipeline Architecture

When any conflict or disruption is introduced, the system executes this pipeline:

```
[ Disruption Event / Incompatible Assignment ]
                       │
                       ▼
            Conflict Detector Process
                       │
         ┌─────────────┴─────────────┐
         ▼                           ▼
[ Hard Constraint Violation ]   [ Soft Constraint Violation ]
  • Immediate Recovery Job        • Objective Score Penalty
  • Priority: EMERGENCY / HIGH    • Logged for optimization
                       │
                       ▼
            Subproblem Isolator
  • Identify directly affected activities
  • Identify ripple-effect activities
  • Freeze all unaffected assignments
                       │
                       ▼
            OS Priority Queue
  • Submit Job (Priority: EMERGENCY, Effective Priority: 0)
                       │
                       ▼
            DSA Incremental Solver
  • BFS Impact Analysis (radius k)
  • Min-Heap Priority Recovery Dispatch
  • CSP Forward Checking & Backtracking
  • In-Building Locality & Minimum-Disruption Scoring
  • Generate 2–3 Feasible Alternatives
                       │
                       ▼
          Simulation Container (DBMS)
  • Save candidate solutions (Active schedule unchanged)
                       │
                       ▼
            Administrator Review
  • Side-by-side alternative comparison card
  • Verify: 0 conflicts, minimal changes
                       │
                       ▼
         ACID Transactional Commit
  • Optimistic Version Validation
  • Commit new ScheduleVersion
  • Dispatch notifications via Transactional Outbox
```

---

## 6. Conflict Severity Matrix

| Severity Level | Impact | System Response |
|---|---|---|
| **CRITICAL** | Room double-booking, faculty contention, division overlap, lab outage | **Blocks timetable publishing.** Triggers automated high-priority recovery job. Cannot be overridden. |
| **HIGH** | Capacity shortfall ($\ge 10$ students), missing mandatory lab equipment | **Blocks timetable publishing.** Must be resolved before schedule activation. |
| **MEDIUM** | Marginal capacity shortfall (1–5 students), inter-building transit violation | **Warning banner.** Generates soft-penalty; administrator can approve with override reason. |
| **LOW** | Faculty preferred slot violation, non-ideal room utilization | **Soft penalty.** Allowed to publish; objective optimizer seeks to minimize. |

---

## 7. Verification & Automated Testing Plan

To guarantee zero regressions in conflict detection:
1. `test_conflict_detector_room_double_booking`: Asserts two classes in same room/slot raise `Class 1` conflict.
2. `test_conflict_detector_shared_faculty`: Asserts faculty teaching in Block A and Block B simultaneously raises `Class 2` conflict.
3. `test_conflict_detector_unavailability`: Asserts outage in Lab A-3 flags exactly the affected sessions.
4. `test_incremental_reschedule_preserves_unaffected`: Asserts that when Lab A-3 is recovered, ECE, MECH, and CIVIL assignments remain unchanged.
5. `test_capacity_check`: Asserts 65-student batch in 40-seat room raises `Class 5` conflict.

