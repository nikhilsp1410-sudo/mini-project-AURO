# AURO — DSA Algorithms Reference (Revised)
## Pre-Built Timetable Validation, Conflict Detection & Minimum-Disruption Recovery

> **Document Type:** Algorithm Design Reference  
> **Version:** 2.0 (Supersedes v1.0 DSATUR generation model)  
> **Status:** ✅ AUTHORITATIVE STANDARD  
> **Core Principle:** AURO uses pre-built university timetables (e.g. KLE Tech benchmark). The DSA engine specializes in **Conflict Detection**, **BFS Impact Analysis**, **Priority Task Scheduling**, and **CSP Recovery with Backtracking & Forward Checking** to deliver minimal-displacement schedule recovery.

---

## 1. Engine Architecture & Component Overview

Rather than attempting to color a blank graph from scratch, the revised DSA Engine models and protects active university operations through 7 distinct components:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        REVISED AURO DSA ENGINE                         │
│                                                                        │
│   1. Conflict Graph           → Resource incompatibility relationships │
│   2. BFS Traversal            → Connected component impact exploration │
│   3. Priority Queue (Min-Heap)→ Urgent disruption job dispatching      │
│   4. Hash Maps                → O(1) in-memory multi-key lookups       │
│   5. CSP Formulation          → Recovery variables, domains & bounds   │
│   6. Backtracking Search      → Recursive candidate search & rollback  │
│   7. Forward Checking         → Lookahead domain pruning on assignment │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
                     Minimum-Disruption Alternatives
                     (Evaluated, Simulated & Ranked)
```

### Finalized 7 DSA Components

| No. | Component | Category | Role in AURO |
|:---:|---|---|---|
| **1** | **Conflict Graph** | Graph Representation | Represents potential incompatibility relationships between timetable activities that share constrained resources. |
| **2** | **BFS (Breadth-First Search)** | Graph Traversal | Explores connected activities from an affected activity node to establish the recovery scope. |
| **3** | **Priority Queue (Min-Heap)** | Priority Queueing | Prioritizes critical emergency recovery jobs before lower-priority tasks. |
| **4** | **Hash Maps** | In-Memory Indexing | Provides $O(1)$ fast lookups for activities, faculty, rooms, and adjacency lists. |
| **5** | **CSP (Constraint Satisfaction)** | Problem Model | Models the affected activities as variables, feasible (Room, Time) as domains, and institutional rules as constraints. |
| **6** | **Backtracking** | Search Algorithm | Systematically searches possible assignments and backtracks when choices lead to dead-ends. |
| **7** | **Forward Checking** | Constraint Propagation | Prunes incompatible options from remaining unassigned variables immediately after an assignment. |

> **Deliberately Excluded Components:**
> - **DSATUR:** Excluded because AURO does not generate the initial timetable from an empty graph.
> - **DFS:** Excluded in favor of level-by-level BFS traversal for localized impact exploration.

---

## 2. Component 1 — Conflict Graph (`graph.py`)

### 2.1 Concept
- **Vertices (Nodes):** Timetable activities $\alpha \in \text{Assignments}$.
- **Edges:** Incompatibility relationships where activities share a mutual resource (same faculty, same student batch, or same room).
- An edge represents a **potential resource contention**. An actual conflict is confirmed when time intervals overlap.

### 2.2 Mathematical Definition
$$G = (V, E)$$
$$V = \{ \alpha_i \mid \alpha_i \text{ is an active timetable assignment} \}$$
$$E = \{ (\alpha_i, \alpha_j) \mid faculty(\alpha_i) = faculty(\alpha_j) \lor batch(\alpha_i) = batch(\alpha_j) \lor room(\alpha_i) = room(\alpha_j) \}$$

### 2.3 Construction Pseudocode
```text
BUILD_CONFLICT_GRAPH(Activities):
    Create empty Graph G
    FOR each activity A in Activities:
        Add vertex A to G

    FOR each resource type R in ['faculty', 'batch', 'room']:
        Group activities by the resource ID they use
        FOR each group G_res:
            FOR each pair (A, B) in G_res:
                IF A != B:
                    Add edge A <-> B labelled with R
    RETURN G
```

### 2.4 Complexity
- Vertex creation: $O(V)$
- Resource grouping with Hash Maps: $O(V)$
- Edge additions: $O(E)$ proportional to shared resource clusters.

---

## 3. Component 2 — Breadth-First Search (`bfs.py`)

### 3.1 Concept
When a disruption event occurs (e.g. Lab A-3 electrical maintenance), BFS starts from the directly impacted activities and traverses neighbor edges level-by-level to map out the ripple-effect cluster.

### 3.2 Pseudocode
```text
BFS_IMPACT_ANALYSIS(Graph, StartActivity):
    Create empty Queue Q
    Create empty Set Visited

    Add StartActivity to Visited
    ENQUEUE(Q, StartActivity)

    WHILE Q is not empty:
        Current = DEQUEUE(Q)
        FOR each Neighbor in Graph.get_neighbors(Current):
            IF Neighbor not in Visited:
                Add Neighbor to Visited
                ENQUEUE(Q, Neighbor)

    RETURN Visited
```

### 3.3 Complexity
- **Time Complexity:** $O(V + E)$
- **Space Complexity:** $O(V)$

---

## 4. Component 3 — Priority Queue (`priority_queue.py`)

### 4.1 Concept
AURO processes recovery operations through a Min-Heap Priority Queue so that urgent classroom emergencies pre-empt background analytics.

### 4.2 Priority Levels
| Level | Trigger Scenario | Priority Rank |
|---|---|:---:|
| **Emergency** | Immediate room/lab outage affecting a class starting today | **1** |
| **Urgent** | Faculty sudden medical leave affecting tomorrow's sessions | **2** |
| **Planned** | Scheduled maintenance submitted days in advance | **3** |
| **Routine** | Timetable import validation or elective balance check | **4** |

### 4.3 Operations & Complexity
- `INSERT(task, priority)`: $O(\log N)$
- `EXTRACT_MIN()`: $O(\log N)$
- `PEEK()`: $O(1)$

---

## 5. Component 4 — Hash Maps (`hash_maps.py`)

### 5.1 Concept
PostgreSQL is the durable source of truth, but during in-memory recovery search, Hash Maps provide immediate $O(1)$ lookup to eliminate redundant DB roundtrips.

```text
faculty_map[faculty_id]       → FacultyProfile
room_map[room_id]             → RoomDetails (capacity, type, equipment)
activity_map[assignment_id]   → TimetableAssignment
adjacency_map[assignment_id]  → Set[ConflictingAssignmentIDs]
division_schedule[(dept, sem, div)] → List[WeeklyAssignments]
```

---

## 6. Component 5 — Constraint Satisfaction Problem (`csp.py`)

### 6.1 Recovery Model
Instead of solving a global CSP for 2,500 classes, AURO constructs a **Localized Recovery CSP**:
- **Variables $X$:** The set of affected activities $\{A_1, A_2, \dots, A_k\}$ identified by the disruption.
- **Domains $D(A_i)$:** Feasible $(Room, TimeSlot)$ pairs filtered by room suitability.
- **Constraints $C$:** Hard non-overlap constraints with each other **AND** non-overlap with fixed unaffected classes.

### 6.2 Half-Open Interval Overlap Definition
Two intervals overlap if and only if:
$$\text{Start}_A < \text{End}_B \;\land\; \text{Start}_B < \text{End}_A$$
Activities ending at 10:00 AM do not overlap activities starting at 10:00 AM.

### 6.3 Hard Constraints (Must NEVER be violated)
1. **Room Exclusivity:** No two activities occupy the same room at overlapping clock times.
2. **Faculty Exclusivity:** A professor cannot teach two activities simultaneously.
3. **Cohort Exclusivity:** A student division cannot attend two classes simultaneously.
4. **Capacity Sufficiency:** $\text{Capacity}(room) \ge \text{StudentCount}(batch)$.
5. **Lab Matching:** Practical courses must use designated laboratories.
6. **Rest Intervals:** Morning Break (10:00–10:30) and Lunch (12:30–01:30) are strictly non-teaching.
7. **Continuous Block Preservation:** 2-hr, 3-hr, and 4-hr lab blocks must not be fragmented.
8. **Fixed Schedule Invariant:** All unaffected classes remain locked.

---

## 7. Components 6 & 7 — Backtracking with Forward Checking (`recovery_solver.py`)

### 7.1 Combined Algorithm
```text
SOLVE_RECOVERY(CSP, Assignment):
    IF all affected variables in CSP.variables are assigned:
        RETURN Assignment  # Valid alternative found

    Var = SELECT_UNASSIGNED_VARIABLE(CSP, Assignment)

    FOR each (Room, Slot) in CSP.domains[Var]:
        IF CONSISTENT(Var, (Room, Slot), Assignment):
            Assignment[Var] = (Room, Slot)
            
            # Forward Checking
            SavedDomains = record_current_domains(CSP)
            Feasible = TRUE
            
            FOR each Neighbor in CSP.get_unassigned_neighbors(Var):
                PRUNE_INCOMPATIBLE_VALUES(Neighbor, (Room, Slot))
                IF CSP.domains[Neighbor] is empty:
                    Feasible = FALSE
                    BREAK
            
            IF Feasible:
                Result = SOLVE_RECOVERY(CSP, Assignment)
                IF Result != FAILURE:
                    RETURN Result
            
            # Backtrack
            Restore_domains(CSP, SavedDomains)
            Delete Assignment[Var]

    RETURN FAILURE
```

---

## 8. Minimum-Disruption Objective & Alternative Ranking

When multiple valid alternatives exist, AURO computes a **Disruption Score ($D$)**:

$$\text{Minimize: } D = w_{room} \cdot \Delta_{\text{rooms}} + w_{time} \cdot \Delta_{\text{times}} + w_{faculty} \cdot \Delta_{\text{faculty}} + w_{students} \cdot N_{\text{students\_affected}}$$

The candidate with the lowest disruption score is ranked as **Alternative 1 (Recommended)** for administrator review.

---

## 9. Verification & Traceability

- **Input Model:** Pre-built timetable sheets (KLE Tech CSE V Sem Div A, B, C, D).
- **Benchmark Outage:** 2-day maintenance of Lab A-3 in Block A.
- **Search Execution:** Re-routes only the 4 affected practical sessions to Lab A-1 / Lab A-5 without touching any of the other 84 divisions.
- **Commit Boundary:** Stored as `SimulationAssignment`, approved by Administrator, committed into `ScheduleVersion 2.0`.
