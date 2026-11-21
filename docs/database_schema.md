# AURO — Database Schema Reference

> **Document Type:** Database Design  
> **Version:** 1.0  
> **Status:** Draft — to be finalized during Phase 1 & 2 implementation

---

## ER Diagram (Text Representation)

```
Department ──< Faculty >── User
     │               │
     │          FacultyAvailability >── TimeSlot
     │
     ├──< StudentBatch
     │
     └──< Course >── CourseOffering ──< Assignment >── ScheduleVersion
                           │                 │
                       Faculty              Room
                       Batch               TimeSlot
                           │
                      CourseOffering
                           │
                   (uses labs if requires_lab)
                        Laboratory ──< Room
                                         │
                                   RoomEquipment >── Equipment

ResourceUnavailability (polymorphic: faculty | room | lab)
Simulation >── SimulationAssignment
AuditLog
Notification
```

---

## Table Definitions

### Department

```sql
CREATE TABLE department (
    department_id   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    department_name VARCHAR(255) NOT NULL,
    department_code VARCHAR(20)  NOT NULL UNIQUE,
    created_at      TIMESTAMPTZ  NOT NULL DEFAULT now()
);
```

---

### User (Application Users)

```sql
CREATE TABLE "user" (
    user_id       UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    full_name     VARCHAR(255) NOT NULL,
    email         VARCHAR(255) NOT NULL UNIQUE,
    password_hash TEXT         NOT NULL,
    role          VARCHAR(50)  NOT NULL CHECK (role IN ('admin','coordinator','faculty','student')),
    department_id UUID         REFERENCES department(department_id) ON DELETE SET NULL,
    is_active     BOOLEAN      NOT NULL DEFAULT TRUE,
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT now()
);

CREATE INDEX idx_user_email ON "user"(email);
CREATE INDEX idx_user_role  ON "user"(role);
CREATE INDEX idx_user_dept  ON "user"(department_id);
```

---

### Faculty

```sql
CREATE TABLE faculty (
    faculty_id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id             UUID NOT NULL UNIQUE REFERENCES "user"(user_id) ON DELETE CASCADE,
    department_id       UUID NOT NULL REFERENCES department(department_id),
    designation         VARCHAR(100),
    maximum_weekly_load INTEGER NOT NULL DEFAULT 20  -- hours per week
);

CREATE INDEX idx_faculty_dept ON faculty(department_id);
```

---

### StudentBatch

```sql
CREATE TABLE student_batch (
    batch_id      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    batch_name    VARCHAR(100) NOT NULL,
    department_id UUID NOT NULL REFERENCES department(department_id),
    semester      INTEGER NOT NULL CHECK (semester BETWEEN 1 AND 10),
    student_count INTEGER NOT NULL CHECK (student_count > 0),
    academic_year VARCHAR(9) NOT NULL   -- e.g. '2025-2026'
);

CREATE INDEX idx_batch_dept ON student_batch(department_id);
```

---

### Course

```sql
CREATE TABLE course (
    course_id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_code       VARCHAR(20)  NOT NULL UNIQUE,
    course_name       VARCHAR(255) NOT NULL,
    department_id     UUID         NOT NULL REFERENCES department(department_id),
    course_type       VARCHAR(50)  NOT NULL CHECK (course_type IN ('lecture','lab','tutorial','seminar')),
    required_duration INTEGER      NOT NULL,  -- minutes per session
    requires_lab      BOOLEAN      NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_course_dept ON course(department_id);
```

---

### Room

```sql
CREATE TABLE room (
    room_id   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    room_name VARCHAR(100) NOT NULL,
    building  VARCHAR(100),
    capacity  INTEGER NOT NULL CHECK (capacity > 0),
    room_type VARCHAR(50) NOT NULL CHECK (room_type IN ('classroom','lab','seminar_hall','auditorium')),
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);

CREATE INDEX idx_room_type   ON room(room_type);
CREATE INDEX idx_room_active ON room(is_active);
```

---

### Laboratory

```sql
CREATE TABLE laboratory (
    laboratory_id   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    room_id         UUID NOT NULL UNIQUE REFERENCES room(room_id) ON DELETE CASCADE,
    laboratory_name VARCHAR(100) NOT NULL,
    laboratory_type VARCHAR(100)  -- e.g. 'computer', 'physics', 'chemistry'
);
```

---

### Equipment

```sql
CREATE TABLE equipment (
    equipment_id   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    equipment_name VARCHAR(255) NOT NULL,
    equipment_type VARCHAR(100),
    quantity       INTEGER NOT NULL DEFAULT 1 CHECK (quantity >= 0)
);
```

---

### RoomEquipment (Junction)

```sql
CREATE TABLE room_equipment (
    room_id            UUID    NOT NULL REFERENCES room(room_id) ON DELETE CASCADE,
    equipment_id       UUID    NOT NULL REFERENCES equipment(equipment_id) ON DELETE CASCADE,
    available_quantity INTEGER NOT NULL DEFAULT 0 CHECK (available_quantity >= 0),
    PRIMARY KEY (room_id, equipment_id)
);
```

---

### TimeSlot

```sql
CREATE TABLE time_slot (
    slot_id     UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    day_of_week INTEGER  NOT NULL CHECK (day_of_week BETWEEN 0 AND 6),  -- 0=Monday
    start_time  TIME     NOT NULL,
    end_time    TIME     NOT NULL,
    UNIQUE (day_of_week, start_time, end_time),
    CHECK (end_time > start_time)
);

CREATE INDEX idx_slot_day ON time_slot(day_of_week);
```

---

### CourseOffering

```sql
CREATE TABLE course_offering (
    offering_id      UUID    PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id        UUID    NOT NULL REFERENCES course(course_id),
    faculty_id       UUID    NOT NULL REFERENCES faculty(faculty_id),
    batch_id         UUID    NOT NULL REFERENCES student_batch(batch_id),
    sessions_per_week INTEGER NOT NULL DEFAULT 1 CHECK (sessions_per_week > 0),
    UNIQUE (course_id, faculty_id, batch_id)
);

CREATE INDEX idx_offering_course  ON course_offering(course_id);
CREATE INDEX idx_offering_faculty ON course_offering(faculty_id);
CREATE INDEX idx_offering_batch   ON course_offering(batch_id);
```

---

### FacultyAvailability

```sql
CREATE TABLE faculty_availability (
    availability_id UUID    PRIMARY KEY DEFAULT gen_random_uuid(),
    faculty_id      UUID    NOT NULL REFERENCES faculty(faculty_id) ON DELETE CASCADE,
    slot_id         UUID    NOT NULL REFERENCES time_slot(slot_id) ON DELETE CASCADE,
    is_available    BOOLEAN NOT NULL DEFAULT TRUE,
    UNIQUE (faculty_id, slot_id)
);

CREATE INDEX idx_favail_faculty ON faculty_availability(faculty_id);
CREATE INDEX idx_favail_slot    ON faculty_availability(slot_id);
```

---

### ResourceUnavailability

```sql
CREATE TABLE resource_unavailability (
    unavailability_id UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    resource_type     VARCHAR(50) NOT NULL CHECK (resource_type IN ('faculty','room','laboratory','equipment')),
    resource_id       UUID        NOT NULL,
    start_datetime    TIMESTAMPTZ NOT NULL,
    end_datetime      TIMESTAMPTZ NOT NULL,
    reason            TEXT,
    created_by        UUID        NOT NULL REFERENCES "user"(user_id),
    created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    CHECK (end_datetime > start_datetime)
);

CREATE INDEX idx_unavail_resource ON resource_unavailability(resource_type, resource_id);
CREATE INDEX idx_unavail_period   ON resource_unavailability(start_datetime, end_datetime);
```

---

### ScheduleVersion

```sql
CREATE TABLE schedule_version (
    schedule_version_id UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    version_number      INTEGER     NOT NULL,
    status              VARCHAR(50) NOT NULL CHECK (status IN ('draft','active','archived','superseded')),
    created_by          UUID        NOT NULL REFERENCES "user"(user_id),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    approved_at         TIMESTAMPTZ,
    description         TEXT
);

CREATE INDEX idx_schedver_status ON schedule_version(status);
CREATE UNIQUE INDEX idx_schedver_active ON schedule_version((1)) WHERE status = 'active';
-- Only one active version at a time
```

---

### Assignment

```sql
CREATE TABLE assignment (
    assignment_id       UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    schedule_version_id UUID        NOT NULL REFERENCES schedule_version(schedule_version_id),
    offering_id         UUID        NOT NULL REFERENCES course_offering(offering_id),
    room_id             UUID        NOT NULL REFERENCES room(room_id),
    slot_id             UUID        NOT NULL REFERENCES time_slot(slot_id),
    session_date_or_day DATE,       -- NULL for recurring weekly assignments
    status              VARCHAR(50) NOT NULL DEFAULT 'assigned'
                        CHECK (status IN ('assigned','cancelled','rescheduled')),
    -- Prevent double-booking: same room + same slot on same day
    UNIQUE (schedule_version_id, room_id, slot_id, session_date_or_day)
);

CREATE INDEX idx_assign_version  ON assignment(schedule_version_id);
CREATE INDEX idx_assign_offering ON assignment(offering_id);
CREATE INDEX idx_assign_room     ON assignment(room_id);
CREATE INDEX idx_assign_slot     ON assignment(slot_id);
```

> **Note:** Additional exclusion constraints using PostgreSQL range types (tsrange) will be added during Phase 2 to prevent clock-time overlaps independent of slot IDs.

---

### Simulation

```sql
CREATE TABLE simulation (
    simulation_id               UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    base_schedule_version_id    UUID        NOT NULL REFERENCES schedule_version(schedule_version_id),
    trigger_unavailability_id   UUID        REFERENCES resource_unavailability(unavailability_id),
    status                      VARCHAR(50) NOT NULL CHECK (status IN ('pending','running','completed','failed','approved')),
    objective_score             FLOAT,
    created_by                  UUID        NOT NULL REFERENCES "user"(user_id),
    created_at                  TIMESTAMPTZ NOT NULL DEFAULT now(),
    completed_at                TIMESTAMPTZ
);

CREATE INDEX idx_sim_base    ON simulation(base_schedule_version_id);
CREATE INDEX idx_sim_status  ON simulation(status);
CREATE INDEX idx_sim_creator ON simulation(created_by);
```

---

### SimulationAssignment

```sql
CREATE TABLE simulation_assignment (
    simulation_assignment_id UUID    PRIMARY KEY DEFAULT gen_random_uuid(),
    simulation_id            UUID    NOT NULL REFERENCES simulation(simulation_id) ON DELETE CASCADE,
    offering_id              UUID    NOT NULL REFERENCES course_offering(offering_id),
    proposed_room_id         UUID    REFERENCES room(room_id),
    proposed_slot_id         UUID    REFERENCES time_slot(slot_id),
    is_changed               BOOLEAN NOT NULL DEFAULT FALSE,
    is_conflict              BOOLEAN NOT NULL DEFAULT FALSE,
    conflict_reason          TEXT
);

CREATE INDEX idx_simassign_sim      ON simulation_assignment(simulation_id);
CREATE INDEX idx_simassign_offering ON simulation_assignment(offering_id);
```

---

### AuditLog

```sql
CREATE TABLE audit_log (
    audit_id        UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID        NOT NULL REFERENCES "user"(user_id),
    action          VARCHAR(100) NOT NULL,
    entity_type     VARCHAR(100) NOT NULL,
    entity_id       UUID,
    previous_values JSONB,
    new_values      JSONB,
    schedule_version_id UUID    REFERENCES schedule_version(schedule_version_id),
    timestamp       TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_audit_user      ON audit_log(user_id);
CREATE INDEX idx_audit_entity    ON audit_log(entity_type, entity_id);
CREATE INDEX idx_audit_timestamp ON audit_log(timestamp DESC);
CREATE INDEX idx_audit_version   ON audit_log(schedule_version_id);
```

---

### Notification

```sql
CREATE TABLE notification (
    notification_id   UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id           UUID        NOT NULL REFERENCES "user"(user_id) ON DELETE CASCADE,
    title             VARCHAR(255) NOT NULL,
    message           TEXT         NOT NULL,
    notification_type VARCHAR(50)  NOT NULL CHECK (notification_type IN (
                          'schedule_change','conflict_alert','approval_required',
                          'simulation_ready','system'
                      )),
    is_read           BOOLEAN      NOT NULL DEFAULT FALSE,
    is_delivered      BOOLEAN      NOT NULL DEFAULT FALSE,
    created_at        TIMESTAMPTZ  NOT NULL DEFAULT now(),
    delivered_at      TIMESTAMPTZ
);

CREATE INDEX idx_notif_user      ON notification(user_id, is_read);
CREATE INDEX idx_notif_delivered ON notification(is_delivered) WHERE is_delivered = FALSE;
```

---

## Indexing Strategy Summary

| Table | Index | Type | Rationale |
|---|---|---|---|
| `user` | `email` | B-tree | Login lookups |
| `user` | `role` | B-tree | RBAC filtering |
| `assignment` | `(schedule_version_id, room_id, slot_id)` | Composite | Conflict checks |
| `assignment` | `offering_id` | B-tree | Timetable retrieval |
| `faculty_availability` | `(faculty_id, slot_id)` | Composite | Availability checks |
| `resource_unavailability` | `(resource_type, resource_id)` | Composite | Resource lookup |
| `audit_log` | `timestamp DESC` | B-tree | Recent activity queries |
| `notification` | `(user_id, is_read)` | Composite | Unread count |
| `notification` | `is_delivered` (partial) | Partial | Outbox polling |
| `schedule_version` | `status = 'active'` (partial unique) | Unique | Single active version |

---

## Concurrency Control

### Optimistic Locking Pattern

```python
# When approving a simulation:
# 1. Read simulation.base_schedule_version_id at approval time
# 2. Lock and read the current active ScheduleVersion
# 3. Assert simulation.base == current active version
# 4. If mismatch → raise 409 Conflict (stale simulation)
# 5. If match → proceed with transaction
```

### Row-Level Locking

```sql
-- Lock the active version row during approval transaction
SELECT * FROM schedule_version
WHERE status = 'active'
FOR UPDATE NOWAIT;
```

---

## Migration Strategy

Managed by **Alembic**:

```
backend/app/db/migrations/
├── env.py
├── script.py.mako
└── versions/
    ├── 001_initial_schema.py
    ├── 002_indexes.py
    ├── 003_constraints.py
    └── ...
```

Run migrations:
```bash
cd backend
alembic upgrade head
```

---

*This document will be updated with the final DDL, ER diagram image, and query analysis results after Phase 1 implementation.*

