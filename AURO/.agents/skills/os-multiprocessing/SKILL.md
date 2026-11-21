---
name: os-multiprocessing
description: Operating Systems engine guidelines for multiprocessing pools, priority queues, aging to prevent starvation, IPC serialization, and deadlock avoidance.
---

# OS Multiprocessing Skill for AURO

## Guidelines
1. **Parallel Execution**:
   - Use Python `multiprocessing` for CPU-intensive DSA algorithms.
   - Do not pass unmanaged database connections across process boundaries.
2. **Priority-Based Scheduling & Aging**:
   - 5 priority classes: Emergency (0), High (1), Medium (2), Normal (3), Low (4).
   - Apply dynamic Aging to prevent starvation of routine background tasks.
3. **IPC Messages**:
   - Clean, serializable dataclasses (`JobMessage`, `ResultMessage`).
4. **Deadlock Prevention**:
   - Use Havender's lexicographical lock acquisition order.
   - Enforce timeouts on all mutex acquisitions.

