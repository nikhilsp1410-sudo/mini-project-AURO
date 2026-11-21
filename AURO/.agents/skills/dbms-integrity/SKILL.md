---
name: dbms-integrity
description: PostgreSQL relational schema design, ACID transactional commits, optimistic version locking, and transactional outbox pattern.
---

# DBMS Integrity Skill for AURO

## Guidelines
1. **Normalization & Integrity**:
   - PostgreSQL as the sole primary persistent storage.
   - Strict foreign keys, non-null constraints, and valid ranges (`capacity > 0`, `end_time > start_time`).
2. **Double-Booking Prevention**:
   - Check actual clock intervals (`start1 < end2 AND start2 < end1`), not merely slot IDs.
3. **Optimistic Version Locking**:
   - Never overwrite active schedules during What-If simulations.
   - When approving, verify active `schedule_version_id` matches the simulation base; reject stale commits with 409 Conflict.
4. **Transactional Outbox**:
   - Insert notifications into `notification_outbox` within the exact same database transaction as the schedule commit.

