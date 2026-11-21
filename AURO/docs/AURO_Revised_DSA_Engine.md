# AURO -- Revised DSA Engine Specification

## Pre-Built Timetable Management, Conflict Detection and Schedule Recovery

**Document purpose:** This document defines the revised Data Structures
and Algorithms (DSA) engine for AURO. It reflects the decision that AURO
will use pre-built university timetables rather than generate the
initial timetable from scratch.

**Core principle:** AURO imports, validates, stores, displays and
manages existing timetables. When a resource or scheduling condition
changes, AURO detects the affected activities and searches for feasible
recovery alternatives while minimizing unnecessary changes.

------------------------------------------------------------------------

## 1. Overview of the revised DSA engine

### 1.1 Previous approach versus revised approach

  -----------------------------------------------------------------------
  Previous approach                   Revised approach
  ----------------------------------- -----------------------------------
  Generate the university timetable   Use pre-built timetables supplied
  from scratch                        by the university or department

  Assign activities to rooms and time Import existing assignments and
  slots as an initial scheduling task validate them

  Use graph colouring / DSATUR to     Do not use DSATUR for initial
  create the initial timetable        timetable generation

  Solve the complete                  Solve only the affected recovery
  timetable-generation problem        problem when changes occur

  Focus on initial allocation         Focus on conflict detection, impact
                                      analysis and minimum-disruption
                                      recovery
  -----------------------------------------------------------------------

AURO does not create the original timetable. The timetable is provided
through an import process, such as a structured CSV or Excel file, or
entered through an authorized interface. AURO maps the imported rows to
its normalized records and validates them before they become an active
schedule version.

### 1.2 Main responsibilities

The revised DSA engine is responsible for:

1.  Reading validated timetable assignments from the database.
2.  Identifying actual timetable conflicts and resource constraints.
3.  Representing relevant relationships between activities.
4.  Identifying activities affected by a resource or availability
    change.
5.  Prioritizing recovery tasks.
6.  Looking up resource and assignment information efficiently.
7.  Finding valid alternative room and time-slot assignments.
8.  Rejecting choices that violate hard constraints.
9.  Comparing feasible alternatives and preferring minimum disruption.
10. Returning proposed alternatives for simulation and approval.

The DSA engine proposes solutions. It must not directly overwrite the
active timetable. The approval and database transaction layer commits an
approved solution.

------------------------------------------------------------------------

## 2. Input data and timetable representation

### 2.1 Source timetable

A timetable may be supplied separately for each:

-   Department
-   Academic year
-   Semester
-   Section or student batch

However, validation must consider the whole university, not just one
department. A faculty member, classroom, laboratory or piece of
equipment may be shared across departments.

### 2.2 Minimum assignment information

Each timetable activity should contain, where applicable:

  -----------------------------------------------------------------------
  Field                               Meaning
  ----------------------------------- -----------------------------------
  assignment_id                       Unique ID for the timetable
                                      assignment

  department_id                       Department offering the activity

  academic_year                       Academic year or year of study

  semester                            Semester number or label

  batch_id                            Student batch or section

  course_id                           Course or subject

  faculty_id                          Assigned faculty member

  room_id                             Assigned classroom or laboratory

  start_datetime                      Actual start date/time or recurring
                                      timetable start time

  end_datetime                        Actual end date/time or recurring
                                      timetable end time

  day_of_week                         Day for a recurring weekly
                                      timetable

  required_room_type                  Classroom, computer lab,
                                      electronics lab, etc.

  required_capacity                   Number of students requiring seats

  required_equipment                  Required equipment or capabilities

  schedule_version_id                 Version to which the assignment
                                      belongs
  -----------------------------------------------------------------------

The exact schema can differ, but the engine needs enough information to
validate time intervals, shared resources, room suitability and the
relevant timetable version.

### 2.3 Important time representation

Do not rely only on labels such as "Slot 1" or "9--10 AM". AURO must be
able to compare actual intervals.

Two intervals overlap if:

``` text
A.start < B.end AND B.start < A.end
```

This treats intervals as half-open: an activity ending at 10:00 does not
overlap one beginning at 10:00.

For recurring weekly schedules, compare the day of week and the
start/end times. For dated events, compare actual date-time intervals.
Handle time zones consistently.

------------------------------------------------------------------------

## 3. Hard constraints and soft objectives

### 3.1 Hard constraints

A proposed recovery assignment must satisfy all applicable hard
constraints:

1.  **Room conflict:** A room cannot host two overlapping activities.
2.  **Faculty conflict:** A faculty member cannot teach two overlapping
    activities.
3.  **Batch conflict:** A student batch cannot attend two overlapping
    activities.
4.  **Capacity:** Room capacity must be at least the required student
    count.
5.  **Room type:** The room must be suitable for the activity.
6.  **Equipment:** All required equipment and capabilities must be
    available.
7.  **Availability:** The room, faculty member and other required
    resources must be available.
8.  **Duration:** The activity must retain its required duration.
9.  **Fixed assignments:** Activities outside the recovery scope should
    remain unchanged unless the authorized recovery policy explicitly
    permits moving them.
10. **Institutional rules:** Any additional university constraints must
    be enforced.

A candidate that violates a hard constraint is not a valid alternative.

### 3.2 Soft objectives

Among alternatives that satisfy every hard constraint, AURO should
prefer the solution that:

-   Changes the fewest existing assignments.
-   Avoids moving unaffected departments or batches.
-   Avoids changing faculty assignments.
-   Avoids moving activities to inconvenient times, where policy permits
    scoring this.
-   Uses suitable rooms without unnecessary resource upgrades.
-   Minimizes the total disruption cost.

A simple objective is to minimize the weighted number of changes:

``` text
Minimize:
    (weight_room   * number_of_room_changes)
  + (weight_time   * number_of_time_changes)
  + (weight_faculty* number_of_faculty_changes)
  + (weight_batch  * number_of_batch_changes)
  + (weight_other  * other_disruption_cost)
```

The weights are policy parameters. They must be chosen and documented by
the project team or university; they should not be presented as
objectively optimal without evaluation.

------------------------------------------------------------------------

## 4. Final DSA components

The revised design uses the following components. They are not all
independent algorithms: some are graph models, some are algorithms, some
are data structures, and CSP is a problem-solving model.

  --------------------------------------------------------------------------------
                    No. Component        Category                 Role in AURO
  --------------------- ---------------- ------------------------ ----------------
                      1 Conflict Graph   Graph representation     Represents
                                                                  relationships
                                                                  between
                                                                  timetable
                                                                  activities that
                                                                  share
                                                                  constrained
                                                                  resources

                      2 BFS              Graph traversal          Explores
                                         algorithm                connected
                                                                  activities from
                                                                  an affected
                                                                  activity

                      3 Priority Queue   Data structure           Processes urgent
                        (Min-Heap or                              recovery jobs
                        configured                                before
                        priority heap)                            lower-priority
                                                                  jobs

                      4 Hash Map         Data structure           Provides fast
                                                                  in-memory lookup
                                                                  of resources,
                                                                  assignments and
                                                                  adjacency
                                                                  information

                      5 CSP              Problem model            Defines recovery
                                                                  variables,
                                                                  domains and
                                                                  constraints

                      6 Backtracking     Search algorithm         Searches
                                                                  possible
                                                                  assignments and
                                                                  reverses choices
                                                                  that lead to
                                                                  failure

                      7 Forward Checking Constraint-propagation   Removes future
                                         technique                choices made
                                                                  invalid by a
                                                                  current
                                                                  assignment
  --------------------------------------------------------------------------------

**DFS is not part of the selected final list.** BFS is selected as the
single graph traversal method for the current design. DSATUR is excluded
because AURO is not colouring a graph to generate the initial timetable.

------------------------------------------------------------------------

# 5. Component 1 -- Conflict Graph

## 5.1 Definition

A graph consists of:

-   **Vertices (nodes):** timetable activities.
-   **Edges:** relationships between activities that share a constrained
    resource or have another relevant incompatibility.

For AURO, each activity is represented by a node. An edge may connect
two activities if they use the same faculty member, student batch, room
or another resource that cannot be shared at overlapping times.

An edge represents a potential incompatibility relationship. It does not
necessarily mean that the two activities currently overlap in time.
Actual conflicts must be confirmed by interval comparison.

## 5.2 Example

Suppose:

  Activity   Faculty   Batch   Room   Time
  ---------- --------- ------- ------ --------------
  A1         F01       CSE-A   R101   09:00--10:00
  A2         F02       ECE-A   R102   09:00--10:00
  A3         F01       CSE-B   R103   10:00--11:00
  A4         F03       CSE-A   R104   10:00--11:00

A1 and A3 share faculty F01, so the graph can represent a faculty
relationship between them. A1 and A4 share batch CSE-A, so it can
represent a batch relationship between them. Whether either pair is an
actual conflict depends on the interval comparison. In this example, the
intervals are adjacent rather than overlapping.

## 5.3 Algorithm: construct a conflict graph

1.  Read the validated activities for the relevant schedule version.
2.  Create one vertex for each activity.
3.  Build indexes of activities by shared resource, such as faculty,
    batch and room.
4.  For each pair sharing a constrained resource, add an edge or
    resource-labelled relationship.
5.  Store the graph using an adjacency list.
6.  Keep resource type information on each edge if useful for explaining
    why the activities are related.

### Pseudocode

``` text
BUILD_CONFLICT_GRAPH(Activities):

    Create empty Graph

    FOR each activity A in Activities:
        Add vertex A to Graph

    FOR each resource type R:
        Group activities by the resource they use

        FOR each group G:
            FOR each pair (A, B) in G:
                IF A is not B:
                    Add relationship A <-> B
                    Label relationship with resource type R

    RETURN Graph
```

A practical implementation should avoid comparing every activity with
every other activity where possible. Grouping activities by resource
makes it possible to create relationships only between activities that
share a relevant resource.

## 5.4 Complexity

-   Creating vertices: O(V)
-   Grouping by resource: approximately O(V) expected with hash-based
    indexes, plus the cost of processing the groups.
-   Creating pairwise relationships within groups: proportional to the
    number of relationships created; a group of k activities can create
    O(k²) pairs.

Here, V is the number of activities.

## 5.5 Role in AURO

The conflict graph provides a structured representation of activity
relationships. It supports impact analysis and BFS traversal. It does
not replace the database's timetable validation or the interval-overlap
checks.

------------------------------------------------------------------------

# 6. Component 2 -- BFS (Breadth-First Search)

## 6.1 Definition

BFS is a graph traversal algorithm that visits nodes level by level,
starting from a selected node.

It uses a **Queue**, which follows FIFO (First In, First Out).

## 6.2 Why AURO uses BFS

When an activity is affected, AURO can start from its graph node and
explore related activities. This can identify a connected group for
impact analysis.

BFS does not itself decide that every visited activity must be
rescheduled. The recovery scope must be determined using resource
relationships, actual conflicts, policy and the reason for the change.

## 6.3 Algorithm

1.  Select the affected activity as the starting node.
2.  Create an empty queue and visited set.
3.  Mark the starting node as visited.
4.  Insert it into the queue.
5.  Remove the front node from the queue.
6.  Examine its adjacent nodes.
7.  For every unvisited adjacent node, mark it visited and add it to the
    queue.
8.  Continue until the queue is empty.
9.  Return the visited activities.

### Pseudocode

``` text
BFS(Graph, Start):

    Create empty Queue
    Create empty Visited set

    Add Start to Visited
    ENQUEUE(Queue, Start)

    WHILE Queue is not empty:

        Current = DEQUEUE(Queue)

        FOR each Neighbour in Graph[Current]:

            IF Neighbour is not in Visited:

                Add Neighbour to Visited
                ENQUEUE(Queue, Neighbour)

    RETURN Visited
```

## 6.4 Example traversal

Suppose the graph has these relationships:

``` text
A1 -- A2
 |
 A3 -- A4
 |
 A5
```

Starting from A1, one possible BFS order is:

``` text
A1 -> A2 -> A3 -> A5 -> A4
```

The exact order depends on the order in which neighbours are stored.

## 6.5 Complexity

With an adjacency-list graph:

-   Time: O(V + E)
-   Space: O(V)

V is the number of vertices and E is the number of edges.

## 6.6 Role in AURO

BFS explores the connected activity relationships around an affected
activity. AURO then checks which activities are truly affected and which
should remain fixed.

------------------------------------------------------------------------

# 7. Component 3 -- Priority Queue

## 7.1 Definition

A Priority Queue stores tasks with priorities. The task with the highest
configured priority is removed first.

AURO can implement it using a heap. If smaller numeric values mean more
urgent tasks, a Min-Heap is appropriate.

## 7.2 Example priorities

  -----------------------------------------------------------------------
  Recovery task                                          Example priority
  ------------------------------ ----------------------------------------
  Immediate faculty or room                                             1
  unavailability affecting an    
  imminent class                 

  Other urgent timetable                                                2
  conflict                       

  Planned resource change                                               3

  Routine timetable update or                                           4
  import validation              
  -----------------------------------------------------------------------

These are example policy values, not universal rules.

## 7.3 Algorithm

1.  Create an empty priority heap.
2.  Receive a recovery task.
3.  Assign its priority according to the configured policy.
4.  Insert the task into the heap.
5.  Remove the highest-priority task.
6.  Process it.
7.  Repeat while tasks remain.

### Pseudocode

``` text
PROCESS_RECOVERY_TASKS(Tasks):

    Create empty PriorityQueue

    FOR each Task in Tasks:
        Priority = CALCULATE_PRIORITY(Task)
        INSERT(PriorityQueue, Priority, Task)

    WHILE PriorityQueue is not empty:

        Task = EXTRACT_HIGHEST_PRIORITY(PriorityQueue)

        PROCESS(Task)
```

If two tasks have equal priority, use a deterministic tie-breaker such
as earlier creation time or task ID.

## 7.4 Complexity

For a binary heap:

  Operation                       Time
  ------------------------- ----------
  Insert                      O(log n)
  Remove highest priority     O(log n)
  Peek                            O(1)

n is the number of queued tasks.

## 7.5 Role in AURO

The Priority Queue orders recovery jobs so that urgent issues are
handled first. It does not decide whether a timetable assignment is
valid and does not itself find a solution.

------------------------------------------------------------------------

# 8. Component 4 -- Hash Map

## 8.1 Definition

A Hash Map stores key-value pairs. A hash function maps a key to an
internal bucket or index, allowing fast average-case access.

## 8.2 Example

``` python
faculty_map = {
    "F01": "Prof. Arun",
    "F02": "Prof. Priya",
    "F03": "Prof. Ravi"
}

room_map = {
    "R101": {
        "capacity": 40,
        "type": "Classroom",
        "projector": True
    },
    "L101": {
        "capacity": 35,
        "type": "Computer Lab",
        "projector": True
    }
}
```

## 8.3 Algorithm: lookup

1.  Receive a key, such as a faculty ID or room ID.
2.  Apply the hash function to the key.
3.  Locate the corresponding bucket.
4.  Compare the stored key with the requested key.
5.  Return the associated value if it matches.
6.  Handle collisions using the chosen Hash Map implementation.

### Pseudocode

``` text
HASH_MAP_LOOKUP(Map, Key):

    Index = HASH(Key)

    FOR each Entry in Map[Index]:

        IF Entry.Key == Key:
            RETURN Entry.Value

    RETURN NOT_FOUND
```

This pseudocode illustrates lookup with bucket collision handling. Real
language Hash Map implementations provide this internally.

## 8.4 Complexity

  Operation     Average   Worst case
  ----------- --------- ------------
  Search           O(1)         O(n)
  Insert           O(1)         O(n)
  Delete           O(1)         O(n)

## 8.5 Role in AURO

Hash Maps can index:

-   Faculty by faculty ID.
-   Rooms and laboratories by resource ID.
-   Activities by assignment ID.
-   Activities by faculty, batch or room.
-   Graph adjacency lists by activity ID.

Hash Maps are in-memory supporting structures. PostgreSQL remains the
permanent source of truth.

------------------------------------------------------------------------

# 9. Component 5 -- CSP (Constraint Satisfaction Problem)

## 9.1 Definition

A Constraint Satisfaction Problem is a mathematical model consisting of:

1.  **Variables:** Items that need values.
2.  **Domains:** Possible values for each variable.
3.  **Constraints:** Rules that valid combinations must satisfy.

CSP is the model of the recovery problem. Backtracking is a search
method that can solve it, and Forward Checking is a technique that
reduces the search space.

## 9.2 AURO example

Suppose Room R101 becomes unavailable and activity A1 needs a new
assignment.

Possible variables:

``` text
X_A1 = new assignment for activity A1
```

Domain:

``` text
Domain(X_A1) = {
    (R102, 09:00-10:00),
    (R103, 09:00-10:00),
    (R102, 10:00-11:00)
}
```

Each domain value is a room-time combination.

Constraints:

-   Room is available.
-   Room capacity is sufficient.
-   Room type is suitable.
-   Required equipment exists.
-   Faculty member is available.
-   Student batch is available.
-   No overlapping room assignment exists.
-   Required activity duration is preserved.

## 9.3 CSP formulation algorithm

1.  Identify the activities that require reassignment.
2.  Create one variable for each affected activity.
3.  Generate possible room-time combinations for each variable.
4.  Remove combinations that violate static constraints, such as
    insufficient room capacity or missing equipment.
5.  Define compatibility constraints between activities.
6.  Keep unaffected assignments fixed unless policy permits them to
    move.
7.  Pass the model to the search procedure.
8.  Validate the complete candidate timetable before returning it.

### Pseudocode

``` text
BUILD_RECOVERY_CSP(AffectedActivities, ExistingSchedule):

    Variables = AffectedActivities

    FOR each Activity in Variables:

        Domain[Activity] = GENERATE_ROOM_TIME_OPTIONS(Activity)

        Remove options that violate:
            Room capacity
            Room type
            Equipment requirements
            Known resource unavailability
            Activity duration rules

    Define constraints for:
        Room overlap
        Faculty overlap
        Batch overlap
        Resource availability
        Fixed unaffected assignments

    RETURN CSP(Variables, Domains, Constraints)
```

## 9.4 Role in AURO

CSP defines exactly what AURO is trying to solve: assign valid room-time
options to affected activities while preserving all hard constraints.

------------------------------------------------------------------------

# 10. Component 6 -- Backtracking

## 10.1 Definition

Backtracking is a recursive search algorithm. It builds a solution one
variable at a time. If a selected value leads to a dead end, the
algorithm removes that assignment and tries another value.

## 10.2 AURO example

Suppose two activities need new assignments:

-   A1 can use R102 or R103.
-   A2 can use R102 or R104.

If A1 and A2 overlap in time, they cannot both use R102.

The solver tries one assignment, checks constraints and continues. If a
later assignment cannot be made, it goes back and tries another choice.

## 10.3 Algorithm

1.  Check whether all affected activities have been assigned.
2.  If yes, return success.
3.  Select an unassigned activity.
4.  Iterate through its remaining domain values.
5.  Check whether the selected value is compatible with existing
    assignments.
6.  If compatible, assign it temporarily.
7.  Recursively assign the next activity.
8.  If the recursive call succeeds, return the solution.
9.  Otherwise, remove the temporary assignment.
10. Try the next domain value.
11. If no value works, return failure.

### Pseudocode

``` text
BACKTRACK(CSP, Assignment):

    IF all variables are assigned:
        RETURN Assignment

    Variable = SELECT_UNASSIGNED_VARIABLE(CSP, Assignment)

    FOR each Value in Domain[Variable]:

        IF CONSISTENT(Variable, Value, Assignment):

            Add Variable = Value to Assignment

            Result = BACKTRACK(CSP, Assignment)

            IF Result is not FAILURE:
                RETURN Result

            Remove Variable from Assignment

    RETURN FAILURE
```

## 10.4 Complexity

If there are n variables and each has at most d possible values, the
worst-case search can be O(d\^n), excluding the cost of checking
constraints.

The actual search can be much smaller when domains are filtered and good
variable/value ordering is used.

## 10.5 Role in AURO

Backtracking searches for valid alternative assignments when an existing
room, time or other resource becomes unavailable.

It should operate on the affected recovery scope, not regenerate the
entire university timetable.

------------------------------------------------------------------------

# 11. Component 7 -- Forward Checking

## 11.1 Definition

Forward Checking is a constraint-propagation technique used during
backtracking.

After assigning a value to one variable, it removes incompatible values
from the domains of unassigned variables.

If any unassigned variable has no remaining values, the current
assignment cannot lead to a complete solution and must be rejected.

## 11.2 AURO example

Suppose:

``` text
A1 = {R102, R103}
A2 = {R102, R104}
```

A1 and A2 overlap in time, so they cannot use the same room.

If AURO assigns:

``` text
A1 = R102
```

Forward Checking removes R102 from A2's domain:

``` text
A2 = {R104}
```

If A2 had no remaining option after filtering, AURO would backtrack and
try another assignment for A1.

## 11.3 Algorithm

1.  Assign a value to the selected variable.
2.  Visit every unassigned variable constrained by it.
3.  Remove domain values incompatible with the new assignment.
4.  If any affected domain becomes empty, report failure.
5.  Otherwise, continue the search.
6.  When backtracking, restore the domains to their previous state.

### Pseudocode

``` text
FORWARD_CHECK(Variable, Value, CSP, Assignment):

    Create a record of removed domain values

    FOR each UnassignedVariable related to Variable:

        FOR each Candidate in Domain[UnassignedVariable]:

            IF Candidate conflicts with (Variable, Value):

                Remove Candidate
                Record the removed Candidate

        IF Domain[UnassignedVariable] is empty:

            Restore all recorded domain values
            RETURN FAILURE

    RETURN SUCCESS
```

In an implementation, domain changes must be restored correctly when the
search backtracks. A trail/stack of removed values is one common
approach.

## 11.4 Complexity

Forward Checking's cost depends on the number of variables, domain sizes
and constraints checked. It adds work at each assignment but can reduce
the number of search branches. It does not remove the exponential
worst-case behaviour of CSP search.

## 11.5 Role in AURO

Forward Checking detects some impossible future assignments early,
reducing wasted search during timetable recovery.

------------------------------------------------------------------------

# 12. How CSP, Backtracking and Forward Checking work together

These three components form the core of AURO's recovery solver.

  -----------------------------------------------------------------------
  Component                           Responsibility
  ----------------------------------- -----------------------------------
  CSP                                 Defines variables, domains and
                                      constraints

  Backtracking                        Searches for an assignment

  Forward Checking                    Removes incompatible future choices
                                      during the search
  -----------------------------------------------------------------------

Combined procedure:

``` text
1. Build the CSP recovery problem.
2. Select an unassigned activity.
3. Try a possible room-time assignment.
4. Check hard constraints.
5. Apply Forward Checking to remaining activities.
6. If a domain becomes empty:
       Undo the assignment.
       Restore removed domain values.
       Try another value.
7. If all activities are assigned:
       Validate the complete candidate schedule.
       Return the feasible solution.
8. If all choices fail:
       Report that no solution was found in the explored search space.
```

A solver can return multiple feasible alternatives by continuing the
search after finding the first solution, subject to a configured search
limit.

------------------------------------------------------------------------

# 13. Complete revised AURO DSA workflow

``` text
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

Hash Maps support lookups throughout the workflow. The database import,
validation, simulation, approval and commit operations are handled by
the corresponding AURO application and database modules.

------------------------------------------------------------------------

# 14. Important validation rules

The DSA engine must not assume that a graph edge alone proves a
conflict. It must check the actual schedule data.

For every candidate assignment, validate:

  Validation          Required check
  ------------------- ---------------------------------------------------
  Room                No overlapping activity in the same room
  Faculty             No overlapping assignment for the faculty member
  Batch               No overlapping activity for the student batch
  Capacity            Capacity is sufficient
  Room type           Required room type is supported
  Equipment           Required equipment is available
  Availability        No resource-unavailability period overlaps
  Duration            Activity duration remains valid
  Fixed assignments   Unaffected assignments remain unchanged
  Schedule version    Candidate is based on the expected active version

The complete candidate timetable must be validated again before it is
approved and committed. This final validation protects against errors in
the recovery search and changes made by other users while a simulation
was running.

------------------------------------------------------------------------

# 15. Minimum-disruption recovery

Finding a valid timetable is not enough. AURO should prefer a valid
timetable that causes the least disruption.

For example, suppose two feasible solutions exist:

  Measure                    Alternative 1   Alternative 2
  ------------------------ --------------- ---------------
  Room changes                           1               2
  Time changes                           0               1
  Faculty changes                        0               0
  Other activity changes                 0               1

Alternative 1 is less disruptive under a simple count-based comparison.

A practical implementation can:

1.  Generate feasible alternatives.
2.  Calculate a disruption score for each.
3.  Rank alternatives from lowest to highest score.
4.  Show the reasons for each change.
5.  Let an authorized administrator select and approve an alternative.

The score should be treated as a ranking preference, not as a hard
constraint. A low-disruption option that violates a hard constraint must
always be rejected.

------------------------------------------------------------------------

# 16. Components deliberately excluded

## 16.1 DSATUR

DSATUR is a graph-colouring heuristic used to assign colours to graph
vertices while trying to avoid adjacent vertices receiving the same
colour.

It is commonly used in timetable generation by treating time slots as
colours.

**Why it is excluded:** AURO uses pre-built timetables and is not using
graph colouring to generate the initial timetable. Its recovery solver
instead models affected assignments as a CSP.

## 16.2 DFS

DFS is a valid graph traversal algorithm, but the current project design
selects BFS as the single traversal algorithm. BFS is sufficient for the
planned level-by-level exploration of connected affected activities.

Do not implement or claim DFS unless the project later introduces a
specific requirement for depth-first exploration.

## 16.3 Full timetable generation

The revised DSA engine does not generate the original timetable for all
departments, years, sections and semesters. It works with the imported
timetable and searches for changes only when recovery is required.

------------------------------------------------------------------------

# 17. Recommended implementation order

Implement and test the DSA engine in this order:

1.  **Timetable data model:** Define activities, resources, time
    intervals and schedule versions.
2.  **Conflict validation:** Implement room, faculty, batch, capacity,
    equipment and availability checks.
3.  **Hash Maps:** Build lookup indexes for assignments and resources.
4.  **Conflict Graph:** Build graph relationships from shared
    constrained resources.
5.  **BFS:** Explore the connected activity group from an affected
    activity.
6.  **Priority Queue:** Order recovery jobs according to priority.
7.  **CSP model:** Define affected variables, possible room-time domains
    and hard constraints.
8.  **Backtracking:** Implement recursive assignment search.
9.  **Forward Checking:** Add domain filtering and correct restoration
    during backtracking.
10. **Minimum-disruption ranking:** Score and rank feasible
    alternatives.
11. **Final validation:** Validate each complete candidate against the
    entire relevant timetable.
12. **Integration:** Send alternatives to simulation and approval;
    commit only after authorization.

Each component should have unit tests and small example datasets before
integration.

------------------------------------------------------------------------

# 18. Testing scenarios

  -----------------------------------------------------------------------
  Test                    Input situation         Expected result
  ----------------------- ----------------------- -----------------------
  T1                      Two activities use the  Conflict detected
                          same room at            
                          overlapping times       

  T2                      Same faculty has        Conflict detected
                          overlapping classes     

  T3                      Same batch has          Conflict detected
                          overlapping classes     

  T4                      Room capacity is too    Candidate rejected
                          small                   

  T5                      Required projector or   Candidate rejected
                          equipment is missing    

  T6                      Room becomes            Affected activity
                          unavailable             identified

  T7                      First room choice       Backtracking tries
                          blocks a later activity another choice

  T8                      A choice removes all    Forward Checking
                          future options          reports failure

  T9                      Multiple recovery jobs  Highest-priority job
                          have different          processed first
                          priorities              

  T10                     Two feasible            Lower-disruption option
                          alternatives have       ranks higher
                          different disruption    

  T11                     Unaffected activities   They remain unchanged
                          exist outside the       
                          recovery scope          

  T12                     Active schedule version Stale simulation is
                          changes after           rejected at commit
                          simulation              
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 19. Final DSA list for project presentation

  ------------------------------------------------------------------------
                           No. DSA component         What it does in AURO
  ---------------------------- --------------------- ---------------------
                             1 Graph Theory --       Represents
                               Conflict Graph        relationships between
                                                     timetable activities

                             2 BFS                   Explores connected
                                                     affected activities

                             3 Priority Queue        Prioritizes recovery
                                                     jobs

                             4 Hash Map              Enables fast lookup
                                                     of resources and
                                                     assignments

                             5 CSP                   Models the recovery
                                                     problem using
                                                     variables, domains
                                                     and constraints

                             6 Backtracking          Searches for feasible
                                                     alternative
                                                     assignments

                             7 Forward Checking      Removes invalid
                                                     future choices during
                                                     search
  ------------------------------------------------------------------------

## Final project statement

**AURO does not generate the university's initial timetable. It uses
pre-built departmental timetables, imports and validates them, detects
timetable and resource conflicts, identifies affected activities, and
applies graph-based impact analysis and constraint-based recovery to
find feasible alternative room and time assignments. It prioritizes
recovery tasks, uses efficient data structures for resource lookup, and
applies CSP with Backtracking and Forward Checking to search for valid
alternatives while minimizing disruption. Proposed changes are simulated
and presented for approval before the active timetable is updated.**

------------------------------------------------------------------------

## 20. Technical accuracy notes

-   A graph edge should represent a defined resource relationship or
    incompatibility, not automatically an actual time conflict.
-   Actual overlap detection must compare time intervals and the
    relevant resource IDs.
-   BFS explores graph connectivity; it does not perform timetable
    optimization.
-   A Priority Queue orders jobs; it does not determine schedule
    feasibility.
-   Hash Maps provide fast in-memory lookup; PostgreSQL remains the
    persistent source of truth.
-   CSP is a problem model. Backtracking is a search algorithm. Forward
    Checking is a propagation technique.
-   Backtracking can find a feasible solution, but minimum disruption
    requires an explicit objective and comparison of candidate
    solutions.
-   A successful partial assignment is not enough. AURO must validate
    the complete candidate timetable before approval.
-   If the search limit is reached, report that the search was
    incomplete; do not incorrectly report that no feasible solution
    exists.
-   Do not claim any component is implemented or benchmarked until it
    has been coded and tested.

**End of document.**
