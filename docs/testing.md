# AURO — Testing Guide

> **Document Type:** Testing Reference  
> **Version:** 1.0  
> **Status:** Draft — to be updated with actual test results

---

## Testing Strategy

AURO uses a multi-layer testing approach aligned with the four engineering domains.

```
┌─────────────────────────────────────────────────────┐
│                 Testing Pyramid                      │
│                                                     │
│           ┌─────────────────────┐                   │
│           │  E2E / Integration  │  ← Few but critical│
│           ├─────────────────────┤                   │
│           │   API Integration   │  ← Scenario-based  │
│           ├─────────────────────┤                   │
│           │     Unit Tests      │  ← Many, fast      │
│           └─────────────────────┘                   │
└─────────────────────────────────────────────────────┘
```

---

## Directory Structure

```
backend/app/tests/
├── conftest.py                  # Shared fixtures
├── unit/
│   ├── dsa/
│   │   ├── test_graph.py
│   │   ├── test_bfs_impact.py
│   │   ├── test_csp.py
│   │   ├── test_backtracking.py
│   │   ├── test_forward_checking.py
│   │   ├── test_priority_queue.py
│   │   ├── test_constraint_validator.py
│   │   ├── test_optimizer.py
│   │   ├── test_incremental_scheduler.py
│   │   └── test_conflict_detector.py
│   ├── os_engine/
│   │   ├── test_coordinator.py
│   │   ├── test_worker_pool.py
│   │   ├── test_job_queue.py
│   │   ├── test_synchronization.py
│   │   └── test_recovery.py
│   └── services/
│       ├── test_schedule_service.py
│       ├── test_simulation_service.py
│       └── test_approval_service.py
├── integration/
│   ├── test_auth_api.py
│   ├── test_resource_crud.py
│   ├── test_schedule_generation.py
│   ├── test_simulation.py
│   ├── test_approval_commit.py
│   ├── test_concurrent_approval.py
│   ├── test_websockets.py
│   ├── test_notifications.py
│   └── test_audit_logs.py
└── performance/
    ├── test_schedule_performance.py
    └── test_api_load.py
```

---

## DSA Unit Tests

### Graph Tests (`test_graph.py`)

```python
class TestConflictGraph:
    
    def test_add_vertex(self):
        """Graph correctly stores activity vertices."""
    
    def test_add_edge_creates_undirected_connection(self):
        """Adding edge (u,v) also creates (v,u)."""
    
    def test_remove_vertex_removes_all_edges(self):
        """Removing a vertex removes all its edges."""
    
    def test_get_degree_correct(self):
        """Degree equals number of adjacent vertices."""
    
    def test_is_adjacent(self):
        """Adjacency check returns correct boolean."""
    
    def test_conflict_edge_faculty_shared(self):
        """Two activities with same faculty produce an edge."""
    
    def test_conflict_edge_batch_shared(self):
        """Two activities with same batch produce an edge."""
    
    def test_no_self_loops(self):
        """An activity is never adjacent to itself."""
    
    def test_conflict_info_contains_reason(self):
        """Edge metadata contains the conflict reason."""
```

---

### BFS Impact Analysis Tests (`test_bfs_impact.py`)

```python
class TestBFSImpact:
    
    def test_direct_neighbors_at_depth_1(self):
        """Activities sharing disrupted resource are discovered at depth 1."""
    
    def test_ripple_effect_at_depth_k(self):
        """Displaced activities propagate cascades up to max_depth k."""
    
    def test_isolated_components_unaffected(self):
        """Unrelated activities outside connected components are excluded."""
    
    def test_cycle_handling_visited_set(self):
        """Cycles in resource graph terminate gracefully using visited set."""
    
    def test_deterministic_traversal_order(self):
        """Same graph and root always produce deterministic BFS layer ordering."""
    
    def test_empty_graph(self):
        """Empty conflict graph produces empty impact set."""
```

---

### CSP Tests (`test_csp.py`)

```python
class TestCSP:
    
    def test_domain_excludes_unavailable_faculty(self):
        """Faculty unavailable in a slot → slot removed from domain."""
    
    def test_domain_excludes_insufficient_capacity(self):
        """Room too small for batch → not in domain."""
    
    def test_domain_excludes_lab_mismatch(self):
        """Lab course cannot be assigned non-lab room."""
    
    def test_domain_respects_resource_unavailability(self):
        """Unavailability record removes (slot, room) from domain."""
    
    def test_hard_constraint_faculty_uniqueness(self):
        """Two activities same faculty + same slot → constraint violated."""
    
    def test_hard_constraint_room_uniqueness(self):
        """Two activities same room + same slot → constraint violated."""
    
    def test_hard_constraint_batch_uniqueness(self):
        """Two activities same batch + same slot → constraint violated."""
    
    def test_soft_constraint_scoring_range(self):
        """Soft constraint score is bounded and comparable."""
    
    def test_hard_constraints_never_traded_for_soft(self):
        """Schedule with hard violation not preferred over lower soft score."""
```

---

### Backtracking Tests (`test_backtracking.py`)

```python
class TestBacktracking:
    
    def test_simple_schedule_found(self):
        """2 non-conflicting activities assigned successfully."""
    
    def test_backtrack_on_dead_end(self):
        """When one choice causes dead-end, correctly backtracks."""
    
    def test_domain_restored_after_backtrack(self):
        """Backtracking restores domains to pre-assignment state."""
    
    def test_no_schedule_returns_none(self):
        """Infeasible problem returns None, not an invalid schedule."""
    
    def test_search_respects_iteration_limit(self):
        """Backtracking stops within configured max iterations."""
    
    def test_complete_schedule_satisfies_all_hard_constraints(self):
        """Every returned schedule satisfies all hard constraints."""
```

---

### Forward Checking Tests (`test_forward_checking.py`)

```python
class TestForwardChecking:
    
    def test_prunes_conflict_neighbor_domains(self):
        """Assigning slot X removes slot X from all conflicting neighbors."""
    
    def test_detects_empty_domain_early(self):
        """Returns failure before attempting further assignments."""
    
    def test_does_not_modify_non_conflicting_domains(self):
        """Non-adjacent activities' domains are unchanged."""
    
    def test_full_restoration_on_backtrack(self):
        """All domain prunings reversed when backtracking."""
```

---

### Incremental Scheduler Tests (`test_incremental_scheduler.py`)

```python
class TestIncrementalScheduler:
    
    def test_identifies_affected_assignments_correctly(self):
        """Only assignments using the unavailable resource are flagged."""
    
    def test_unaffected_assignments_preserved(self):
        """Non-affected assignments remain identical in output."""
    
    def test_generates_valid_alternative_schedule(self):
        """Output satisfies all hard constraints."""
    
    def test_minimizes_changed_assignments(self):
        """Fewer changes preferred over more changes."""
    
    def test_reports_unresolved_conflicts(self):
        """When no alternative exists, conflict is explicitly reported."""
    
    def test_does_not_modify_active_schedule(self):
        """Input active schedule object is not mutated."""
    
    def test_multiple_alternatives_ranked(self):
        """When multiple solutions exist, returns ranked list."""
```

---

## DBMS Integration Tests

### Constraint Tests (`test_resource_crud.py`)

```python
class TestDatabaseConstraints:
    
    def test_unique_department_code(self):
        """Creating department with duplicate code raises integrity error."""
    
    def test_unique_user_email(self):
        """Registering with existing email fails at DB level."""
    
    def test_foreign_key_faculty_department(self):
        """Faculty cannot reference non-existent department."""
    
    def test_room_capacity_positive(self):
        """Room with capacity ≤ 0 is rejected."""
    
    def test_time_slot_end_after_start(self):
        """Time slot with end ≤ start is rejected."""
```

---

### Concurrency Tests (`test_concurrent_approval.py`)

```python
class TestConcurrentApproval:
    
    def test_only_one_approval_succeeds(self):
        """
        Two simultaneous approvals of the same simulation:
        - One succeeds
        - The other receives 409 Conflict
        """
    
    def test_stale_simulation_rejected(self):
        """
        Simulation based on version V2:
        If V3 is now active → approval rejected with 409
        """
    
    def test_rollback_on_constraint_violation(self):
        """
        If hard constraint violated during commit:
        - Transaction rolled back
        - No new ScheduleVersion created
        - No notifications sent
        """
    
    def test_audit_log_created_on_commit(self):
        """Successful commit creates audit log record."""
    
    def test_notification_created_inside_transaction(self):
        """Notifications appear only after successful commit."""
```

---

## OS Engine Tests

### Worker Pool Tests (`test_worker_pool.py`)

```python
class TestWorkerPool:
    
    def test_workers_start_on_initialization(self):
        """Pool with N workers creates N active processes."""
    
    def test_job_dispatched_to_available_worker(self):
        """Job appears in worker's queue."""
    
    def test_failed_worker_detected(self):
        """Health monitor identifies a crashed worker."""
    
    def test_job_requeued_after_worker_crash(self):
        """Job from crashed worker re-enters the queue."""
    
    def test_retry_limit_enforced(self):
        """Job exceeding max retries is marked FAILED, not requeued."""
    
    def test_graceful_shutdown(self):
        """All workers complete current jobs before shutting down."""
    
    def test_job_timeout_detected(self):
        """Job running too long is cancelled and worker recycled."""
```

---

### Priority and Aging Tests (`test_job_queue.py`)

```python
class TestJobQueue:
    
    def test_emergency_job_before_normal(self):
        """EMERGENCY priority job dequeued before NORMAL."""
    
    def test_aging_increases_effective_priority(self):
        """NORMAL job waiting > aging_threshold promoted in queue."""
    
    def test_no_starvation(self):
        """LOW priority job eventually dequeued after aging."""
    
    def test_ipc_job_arrives_in_worker_queue(self):
        """Job pushed by coordinator appears in worker's multiprocessing.Queue."""
```

---

## Web / API Tests

### Authentication Tests (`test_auth_api.py`)

```python
class TestAuthentication:
    
    def test_login_success_returns_token(self):
        """Valid credentials return JWT token."""
    
    def test_login_wrong_password_401(self):
        """Wrong password returns 401."""
    
    def test_protected_endpoint_without_token_401(self):
        """Missing Authorization header returns 401."""
    
    def test_expired_token_401(self):
        """Expired JWT returns 401."""
    
    def test_admin_can_register_user(self):
        """Admin role can create new users."""
    
    def test_student_cannot_register_user(self):
        """Student role gets 403 on register endpoint."""
```

---

### RBAC Tests (embedded in route tests)

```python
class TestRBAC:
    
    def test_student_cannot_generate_schedule(self):
        """POST /schedules/generate returns 403 for student token."""
    
    def test_faculty_cannot_approve_simulation(self):
        """POST /simulations/{id}/approve returns 403 for faculty token."""
    
    def test_coordinator_can_report_unavailability(self):
        """POST /unavailability returns 201 for coordinator token."""
    
    def test_admin_can_delete_department(self):
        """DELETE /departments/{id} succeeds for admin."""
    
    def test_student_can_view_own_timetable(self):
        """GET /schedules/{id}?batch_id=... returns 200 for student."""
```

---

### Schedule Generation Tests (`test_schedule_generation.py`)

```python
class TestScheduleGeneration:
    
    def test_generation_job_created(self):
        """POST /schedules/generate returns job_id with status QUEUED."""
    
    def test_job_completes_within_timeout(self):
        """Job reaches COMPLETED status within 10 seconds for 100 activities."""
    
    def test_schedule_satisfies_hard_constraints(self):
        """GET /schedules/{id}/conflicts returns zero hard conflicts."""
    
    def test_infeasible_problem_reported_not_silent(self):
        """When no valid schedule exists, job FAILED with descriptive error."""
    
    def test_websocket_progress_events_received(self):
        """WebSocket client receives JOB_PROGRESS events during generation."""
```

---

### End-to-End: Lab 3 Unavailability Scenario

```python
class TestLabUnavailabilityE2E:
    """
    Complete end-to-end test of the core AURO scenario.
    """
    
    def test_01_initial_schedule_active(self):
        """Active schedule version exists with Lab 3 assignments."""
    
    def test_02_report_lab3_unavailability(self):
        """POST /unavailability creates record and triggers recovery job."""
    
    def test_03_conflict_detection_job_created(self):
        """Job of type CONFLICT_DETECTION created with HIGH priority."""
    
    def test_04_simulation_completed(self):
        """Simulation reaches 'completed' status."""
    
    def test_05_alternatives_generated(self):
        """GET /simulations/{id}/alternatives returns at least 1 alternative."""
    
    def test_06_active_schedule_unchanged_during_simulation(self):
        """Original active schedule version and assignments are identical."""
    
    def test_07_alternative_satisfies_hard_constraints(self):
        """Each alternative has zero hard constraint violations."""
    
    def test_08_alternatives_ranked_by_objective(self):
        """Alternatives ordered: fewer changes + higher score first."""
    
    def test_09_stale_simulation_rejected(self):
        """Approving with wrong active version returns 409."""
    
    def test_10_approval_commit_succeeds(self):
        """POST /simulations/{id}/commit returns new schedule version."""
    
    def test_11_new_version_is_active(self):
        """GET /schedules shows new version with status 'active'."""
    
    def test_12_previous_version_superseded(self):
        """Old active version now has status 'superseded'."""
    
    def test_13_audit_log_created(self):
        """GET /audit shows SCHEDULE_COMMITTED action."""
    
    def test_14_notifications_sent(self):
        """Affected faculty and students have unread notifications."""
    
    def test_15_websocket_schedule_updated_event(self):
        """WebSocket clients received SCHEDULE_UPDATED event."""
```

---

## Running Tests

### Setup

```bash
cd backend

# Install test dependencies
pip install pytest pytest-asyncio pytest-cov httpx

# Run all tests
pytest app/tests/ -v

# Run with coverage
pytest app/tests/ --cov=app --cov-report=html

# Run only DSA unit tests
pytest app/tests/unit/dsa/ -v

# Run only integration tests
pytest app/tests/integration/ -v

# Run the E2E scenario test
pytest app/tests/integration/test_e2e_lab_scenario.py -v
```

### Test Database

Tests use a **separate PostgreSQL test database** configured via:

```
TEST_DATABASE_URL=postgresql://auro_test:password@localhost:5432/auro_test
```

Each test session runs migrations on a fresh schema and tears it down after.

### Environment

```bash
# .env.test
DATABASE_URL=postgresql://auro_test:password@localhost:5432/auro_test
JWT_SECRET_KEY=test-secret-key-not-for-production
ENVIRONMENT=test
```

---

## Performance Benchmarks

> These targets are acceptance criteria and must be **measured, not assumed**.

| Benchmark | Target | Method |
|---|---|---|
| Schedule generation (100 activities) | < 10 seconds | `pytest-benchmark` |
| Schedule generation (500 activities) | < 60 seconds | `pytest-benchmark` |
| Zero hard constraint violations | 100% | Constraint validator |
| P95 API read response | < 500ms | `locust` load test |
| Simulation creation | < 2 seconds | Integration test |
| Concurrent approval conflict detection | 100% rejection | Concurrent test |

---

*Test results and coverage reports will be added to `docs/test_results/` as each phase is completed.*

