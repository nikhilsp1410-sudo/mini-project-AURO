# AURO — API Documentation

> **Document Type:** API Reference  
> **Version:** 1.0  
> **Base URL:** `http://localhost:8000/api/v1`  
> **Authentication:** Bearer JWT token (except `/auth/login`)

---

## Authentication

All protected endpoints require the header:
```
Authorization: Bearer <jwt_token>
```

WebSocket connections use query param:
```
ws://localhost:8000/ws?token=<jwt_token>
```

---

## Endpoints

### Authentication

#### POST `/auth/login`
Login and receive a JWT token.

**Request:**
```json
{
  "email": "admin@university.edu",
  "password": "securepassword"
}
```

**Response `200`:**
```json
{
  "access_token": "eyJ...",
  "token_type": "bearer",
  "user": {
    "user_id": "uuid",
    "full_name": "Admin User",
    "email": "admin@university.edu",
    "role": "admin"
  }
}
```

**Errors:** `401` Invalid credentials | `422` Validation error

---

#### POST `/auth/register`
Register a new user. *(Admin only)*

**Request:**
```json
{
  "full_name": "Dr. Jane Smith",
  "email": "jane.smith@university.edu",
  "password": "securepassword",
  "role": "faculty",
  "department_id": "uuid"
}
```

**Response `201`:** Created user object (without password_hash)

---

#### GET `/auth/me`
Get the authenticated user's profile.

**Response `200`:**
```json
{
  "user_id": "uuid",
  "full_name": "...",
  "email": "...",
  "role": "admin",
  "department_id": "uuid",
  "is_active": true
}
```

---

### Departments

#### GET `/departments`
List all departments. *(All authenticated roles)*

**Query Params:** `skip=0`, `limit=100`

**Response `200`:** `Array<Department>`

---

#### POST `/departments`
Create a department. *(Admin only)*

**Request:**
```json
{
  "department_name": "Computer Science",
  "department_code": "CS"
}
```

**Response `201`:** Created `Department`

---

#### PUT `/departments/{department_id}`
Update a department. *(Admin only)*

#### DELETE `/departments/{department_id}`
Delete a department. *(Admin only)*

---

### Faculty

#### GET `/faculty`
List faculty. *(Admin, Coordinator)*

**Query Params:** `department_id`, `skip`, `limit`

**Response `200`:** `Array<FacultyDetail>`

---

#### POST `/faculty`
Create faculty member. *(Admin only)*

**Request:**
```json
{
  "user_id": "uuid",
  "department_id": "uuid",
  "designation": "Associate Professor",
  "maximum_weekly_load": 18
}
```

---

#### GET `/faculty/{faculty_id}/availability`
Get faculty availability schedule.

#### PUT `/faculty/{faculty_id}/availability`
Update faculty availability. *(Admin, Coordinator)*

**Request:**
```json
{
  "availability": [
    { "slot_id": "uuid", "is_available": true },
    { "slot_id": "uuid", "is_available": false }
  ]
}
```

---

### Rooms

#### GET `/rooms`
List rooms. *(All authenticated)*

**Query Params:** `room_type`, `min_capacity`, `is_active`

#### POST `/rooms`
Create room. *(Admin only)*

**Request:**
```json
{
  "room_name": "CS Lab 3",
  "building": "Tech Block",
  "capacity": 40,
  "room_type": "lab",
  "is_active": true
}
```

#### PUT `/rooms/{room_id}`
Update room. *(Admin only)*

---

### Laboratories

#### GET `/laboratories`
List all laboratories with their room details.

#### POST `/laboratories`
Create laboratory entry for an existing room. *(Admin only)*

**Request:**
```json
{
  "room_id": "uuid",
  "laboratory_name": "Network Lab",
  "laboratory_type": "computer"
}
```

---

### Equipment

#### GET `/equipment`
List all equipment.

#### POST `/equipment`
Create equipment record. *(Admin only)*

#### POST `/rooms/{room_id}/equipment`
Assign equipment to a room. *(Admin only)*

**Request:**
```json
{
  "equipment_id": "uuid",
  "available_quantity": 20
}
```

---

### Courses

#### GET `/courses`
List courses. *(All authenticated)*

**Query Params:** `department_id`, `requires_lab`, `course_type`

#### POST `/courses`
Create course. *(Admin, Coordinator)*

**Request:**
```json
{
  "course_code": "CS301",
  "course_name": "Operating Systems",
  "department_id": "uuid",
  "course_type": "lecture",
  "required_duration": 60,
  "requires_lab": false
}
```

---

### Student Batches

#### GET `/batches`
List student batches.

#### POST `/batches`
Create student batch. *(Admin, Coordinator)*

**Request:**
```json
{
  "batch_name": "CS-2024-A",
  "department_id": "uuid",
  "semester": 5,
  "student_count": 60,
  "academic_year": "2025-2026"
}
```

---

### Course Offerings

#### GET `/offerings`
List course offerings (faculty-batch-course assignments).

#### POST `/offerings`
Create course offering. *(Admin, Coordinator)*

**Request:**
```json
{
  "course_id": "uuid",
  "faculty_id": "uuid",
  "batch_id": "uuid",
  "sessions_per_week": 3
}
```

---

### Resource Unavailability

#### POST `/unavailability`
Report resource unavailability. *(Admin, Coordinator)*

**Request:**
```json
{
  "resource_type": "laboratory",
  "resource_id": "uuid-of-lab-3",
  "start_datetime": "2025-11-10T00:00:00Z",
  "end_datetime": "2025-11-12T23:59:59Z",
  "reason": "Scheduled maintenance"
}
```

**Response `201`:** Created unavailability record. Triggers conflict detection job.

#### GET `/unavailability`
List all unavailability records. *(Admin, Coordinator)*

---

### Schedules

#### POST `/schedules/generate`
Trigger schedule generation. *(Admin only)*

**Request:**
```json
{
  "department_id": "uuid",
  "semester": 5,
  "academic_year": "2025-2026",
  "config": {
    "max_iterations": 10000,
    "objective_weights": {
      "faculty_preference": 0.3,
      "gap_penalty": 0.2,
      "utilization": 0.2,
      "stability": 0.3
    }
  }
}
```

**Response `202`:** Job created.
```json
{
  "job_id": "uuid",
  "status": "QUEUED",
  "message": "Schedule generation job queued"
}
```

---

#### GET `/schedules`
List all schedule versions.

**Response `200`:** `Array<ScheduleVersion>`

---

#### GET `/schedules/{schedule_version_id}`
Get a schedule version with its assignments.

**Response `200`:**
```json
{
  "schedule_version_id": "uuid",
  "version_number": 3,
  "status": "active",
  "assignments": [
    {
      "assignment_id": "uuid",
      "offering_id": "uuid",
      "room": { "room_id": "...", "room_name": "..." },
      "slot": { "slot_id": "...", "day_of_week": 0, "start_time": "09:00", "end_time": "10:00" },
      "status": "assigned"
    }
  ]
}
```

---

#### GET `/schedules/{schedule_version_id}/conflicts`
Get detected conflicts in a schedule version.

**Response `200`:**
```json
{
  "conflict_count": 2,
  "conflicts": [
    {
      "conflict_type": "faculty_overlap",
      "severity": "critical",
      "affected_faculty": { "faculty_id": "...", "full_name": "..." },
      "assignments": ["uuid1", "uuid2"],
      "slot": { "day_of_week": 1, "start_time": "10:00" }
    }
  ]
}
```

---

### Jobs

#### GET `/jobs/{job_id}`
Get job status and progress.

**Response `200`:**
```json
{
  "job_id": "uuid",
  "job_type": "SCHEDULE_GENERATION",
  "status": "RUNNING",
  "priority": 3,
  "progress": 45,
  "created_at": "...",
  "started_at": "...",
  "worker_id": "worker-2",
  "result_summary": null,
  "error": null
}
```

---

#### POST `/jobs/{job_id}/cancel`
Cancel a pending or running job. *(Admin only)*

**Response `200`:**
```json
{
  "job_id": "uuid",
  "status": "CANCELLED"
}
```

---

### Simulations

#### POST `/simulations`
Create a what-if simulation for a resource unavailability.

**Request:**
```json
{
  "unavailability_id": "uuid",
  "base_schedule_version_id": "uuid",
  "config": {
    "max_alternatives": 3,
    "objective_weights": { ... }
  }
}
```

**Response `202`:**
```json
{
  "simulation_id": "uuid",
  "job_id": "uuid",
  "status": "pending"
}
```

---

#### GET `/simulations/{simulation_id}`
Get simulation details and status.

**Response `200`:**
```json
{
  "simulation_id": "uuid",
  "status": "completed",
  "base_schedule_version_id": "uuid",
  "affected_assignments_count": 4,
  "alternatives_count": 3,
  "created_at": "..."
}
```

---

#### GET `/simulations/{simulation_id}/alternatives`
Get generated alternative schedules with comparison metrics.

**Response `200`:**
```json
{
  "simulation_id": "uuid",
  "alternatives": [
    {
      "alternative_id": "uuid",
      "rank": 1,
      "objective_score": 87.5,
      "changed_assignments": 2,
      "unresolved_conflicts": 0,
      "room_utilization": 0.78,
      "faculty_preference_score": 0.85,
      "assignments": [ ... ]
    }
  ]
}
```

---

### Approval

#### POST `/simulations/{simulation_id}/approve`
Mark a simulation alternative as selected for approval. *(Admin only)*

**Request:**
```json
{
  "alternative_id": "uuid",
  "current_active_version_id": "uuid"
}
```

**Response `200`:**
```json
{
  "simulation_id": "uuid",
  "alternative_id": "uuid",
  "status": "approved",
  "version_check": "passed"
}
```

**Errors:**
- `403` Unauthorized role
- `409` Active version mismatch (stale simulation)
- `422` Hard constraint violation in proposed changes

---

#### POST `/simulations/{simulation_id}/commit`
Transactionally apply the approved alternative. *(Admin only)*

**Request:**
```json
{
  "simulation_id": "uuid",
  "approved_alternative_id": "uuid"
}
```

**Response `200`:**
```json
{
  "new_schedule_version_id": "uuid",
  "version_number": 4,
  "status": "active",
  "audit_log_id": "uuid",
  "notifications_queued": 12
}
```

**Errors:**
- `403` Unauthorized
- `409` Concurrent modification detected
- `500` Transaction failed (automatically rolled back)

---

### Notifications

#### GET `/notifications`
Get notifications for the authenticated user.

**Query Params:** `is_read=false`, `skip=0`, `limit=20`

**Response `200`:**
```json
{
  "unread_count": 3,
  "notifications": [
    {
      "notification_id": "uuid",
      "title": "Schedule Updated",
      "message": "Your OS Lab session on Monday has been moved to Tuesday.",
      "notification_type": "schedule_change",
      "is_read": false,
      "created_at": "..."
    }
  ]
}
```

---

#### PATCH `/notifications/{notification_id}/read`
Mark a notification as read.

**Response `200`:** `{ "is_read": true }`

---

### Audit Logs

#### GET `/audit`
Retrieve audit logs. *(Admin only)*

**Query Params:** `user_id`, `entity_type`, `action`, `start_date`, `end_date`, `skip`, `limit`

**Response `200`:**
```json
{
  "total": 150,
  "logs": [
    {
      "audit_id": "uuid",
      "user": { "user_id": "...", "full_name": "...", "role": "admin" },
      "action": "SCHEDULE_COMMITTED",
      "entity_type": "schedule_version",
      "entity_id": "uuid",
      "previous_values": { "status": "draft" },
      "new_values": { "status": "active", "version_number": 4 },
      "timestamp": "2025-11-10T14:32:00Z"
    }
  ]
}
```

---

## WebSocket Events

### Connection

```
ws://localhost:8000/ws?token=<jwt_token>
```

### Event Format

```json
{
  "event": "JOB_PROGRESS",
  "data": {
    "job_id": "uuid",
    "status": "RUNNING",
    "progress": 72,
    "message": "Assigning activities 72/100..."
  },
  "timestamp": "2025-11-10T14:32:00Z"
}
```

### Event Types

| Event | Description | Audience |
|---|---|---|
| `JOB_PROGRESS` | Job progress update (0-100%) | Job creator |
| `JOB_COMPLETED` | Job finished successfully | Job creator |
| `JOB_FAILED` | Job failed with error | Job creator |
| `SIMULATION_READY` | Simulation alternatives available | Simulation creator |
| `SCHEDULE_UPDATED` | New active schedule version | All users |
| `NOTIFICATION` | New notification | Specific user |
| `WORKER_STATUS` | Worker pool status | Admins |
| `CONFLICT_DETECTED` | New conflict found | Admins, Coordinators |

---

## HTTP Status Codes

| Code | Meaning | Usage |
|---|---|---|
| `200` | OK | Successful GET, PATCH |
| `201` | Created | Successful POST (new resource) |
| `202` | Accepted | Job queued asynchronously |
| `400` | Bad Request | Invalid input format |
| `401` | Unauthorized | Missing or invalid JWT |
| `403` | Forbidden | Insufficient role |
| `404` | Not Found | Resource does not exist |
| `409` | Conflict | Version mismatch or concurrent modification |
| `422` | Unprocessable Entity | Validation error (Pydantic) |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Internal Server Error | Unexpected error |

---

## Error Response Format

```json
{
  "detail": "Human-readable error message",
  "error_code": "STALE_SIMULATION",
  "context": {
    "expected_version": "uuid-v3",
    "current_version": "uuid-v4"
  }
}
```

---

*Full interactive API documentation available at `http://localhost:8000/docs` (Swagger UI) and `http://localhost:8000/redoc` (ReDoc) when the backend is running.*

