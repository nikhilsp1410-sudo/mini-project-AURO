---
name: dsa-orchestration
description: Revised DSA Engine guidelines for pre-built timetable validation, Conflict Graph modeling, BFS impact exploration, Priority Queue task dispatching, Hash Map lookups, and CSP Backtracking with Forward Checking for minimum-disruption recovery.
---

# DSA Orchestration Skill for AURO (Revised)

## Core Guidelines

1. **Pre-Built Timetable Ingestion**:
   - AURO does NOT generate timetables from scratch. Pre-built departmental timetables (e.g. KLE Tech CSE Sem V Divisions A–D) are imported and validated as ground truth.
   - Do NOT use DSATUR or full-slate graph coloring.

2. **The Seven DSA Components**:
   - **Conflict Graph**: Represents resource-sharing relationships (shared faculty, batch, room) between activities.
   - **BFS (Breadth-First Search)**: Explores the ripple-effect cluster of activities connected to an affected resource.
   - **Priority Queue (Min-Heap)**: Dispatches recovery jobs based on operational urgency (Emergency = 1, High = 2, Planned = 3, Routine = 4).
   - **Hash Maps**: Delivers $O(1)$ fast in-memory indexing of faculty, rooms, assignments, and adjacency lists.
   - **CSP (Constraint Satisfaction Problem)**: Models the localized recovery subproblem (variables = affected activities, domains = feasible room-slot options).
   - **Backtracking**: Recursively searches valid recovery assignments and rolls back on dead-ends.
   - **Forward Checking**: Lookahead domain pruning that removes conflicting options from unassigned neighbors immediately upon assignment.

3. **Time Representation (Half-Open Intervals)**:
   - Always compare clock intervals: `start1 < end2 AND start2 < end1`.
   - Never rely solely on slot label strings.

4. **Minimum Disruption Recovery**:
   - Keep unaffected classes strictly locked in place.
   - Rank feasible recovery alternatives by minimum disruption score:
     $$D = w_{room} \Delta_{rooms} + w_{time} \Delta_{times} + w_{faculty} \Delta_{faculty}$$
