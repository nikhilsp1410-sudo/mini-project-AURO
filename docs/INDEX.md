# AURO Documentation Index & Specification Blueprint

> **Last Updated:** 2026-11-11  
> **Status:** Architecture & Specifications Stage (Clean Workspace Ready for Implementation)

---

## ⚠️ Authoritative Specifications

1. **[`AURO_Revised_DSA_Engine.md`](./AURO_Revised_DSA_Engine.md)** — **Authoritative Revised DSA Specification** (Pre-built timetable management, conflict detection, BFS impact analysis, Priority Queue, Hash Maps, CSP Backtracking & Forward Checking for minimum-disruption recovery).
2. **[`COMPLETE_SPECIFICATION.md`](./COMPLETE_SPECIFICATION.md)** — **Single Source of Truth** for core engine rules, system architecture, and module contracts.
3. **[`KLE_TECH_REFERENCE_TIMETABLE_SPECIFICATION.md`](./KLE_TECH_REFERENCE_TIMETABLE_SPECIFICATION.md)** — **Authoritative Academic Reference Timetable** (KLE Tech CSE V Sem Divisions A–D: Exact slots, sub-batch splitting, co-faculty lab staffing, continuous multi-hour lab blocks, and compulsory generation rules).
4. **[`ENGINEERING_CAMPUS_SPECIFICATION.md`](./ENGINEERING_CAMPUS_SPECIFICATION.md)** — **4-Department Campus Blueprint** (4 separate buildings, 4 academic years, 5–6 divisions per year, curriculum taxonomy).
5. **[`CONFLICT_MODELS_AND_SCENARIOS.md`](./CONFLICT_MODELS_AND_SCENARIOS.md)** — **Conflict Engineering & Failure Scenarios** (The 7 Conflict Classes, mathematical invariants, cross-department sharing contention, and 5 standard demo scenarios).

---

## Document Registry

| Document | Description | Status |
|---|---|---|
| [**AURO_Revised_DSA_Engine.md**](./AURO_Revised_DSA_Engine.md) | Revised DSA: Pre-built timetables, Conflict Graph, BFS, Priority Queue, CSP | ✅ FINAL |
| [**COMPLETE_SPECIFICATION.md**](./COMPLETE_SPECIFICATION.md) | Core specification: 4-engine responsibilities, DSA, DBMS, OS, Web | ✅ FINAL |
| [**KLE_TECH_REFERENCE_TIMETABLE_SPECIFICATION.md**](./KLE_TECH_REFERENCE_TIMETABLE_SPECIFICATION.md) | Real-world academic benchmark: CSE V Sem Divs A-D, co-faculty, lab blocks | ✅ Active |
| [**ENGINEERING_CAMPUS_SPECIFICATION.md**](./ENGINEERING_CAMPUS_SPECIFICATION.md) | Institutional blueprint: 4 departments, 4 blocks, ~88 divisions | ✅ Active |
| [**CONFLICT_MODELS_AND_SCENARIOS.md**](./CONFLICT_MODELS_AND_SCENARIOS.md) | Conflict taxonomy: 7 conflict classes, invariants, demo scenarios | ✅ Active |
| [MASTER_PROMPT.md](./MASTER_PROMPT.md) | Original prompt (v1.0) — reference only | 📚 Reference |
| [ADDITIONAL_DETAILS.md](./ADDITIONAL_DETAILS.md) | Supplementary details — resolved | ✅ Resolved |
| [architecture.md](./architecture.md) | 5-layer architecture, diagrams, scheduling workflows | ✅ Draft |
| [database_schema.md](./database_schema.md) | ER design, all 19 relational table DDLs, indexing strategy | ✅ Draft |
| [algorithms.md](./algorithms.md) | DSA engine design: Conflict Graph, BFS, Priority Queue, Hash Maps, CSP | ✅ FINAL |
| [api_documentation.md](./api_documentation.md) | REST API endpoints and WebSocket event specifications | ✅ Draft |
| [testing.md](./testing.md) | Test plan, test cases, E2E scenario (Lab 3 outage), benchmarks | ✅ Draft |

---

## The 7 Invariant Conflict Classes

| Conflict Class | Invariant Name | Real-World Trigger | Severity |
|:---:|---|---|:---:|
| **Class 1** | Room Double-Booking | Two classes placed in same room during overlapping times | **CRITICAL** |
| **Class 2** | Faculty Contention | Same professor booked for 2 classes across cohorts/blocks | **CRITICAL** |
| **Class 3** | Student Division Overlap | Same division scheduled for 2 simultaneous lectures/labs | **CRITICAL** |
| **Class 4** | Resource Unavailability | Room/Lab/Faculty outage (e.g. Lab A-3 2-day maintenance) | **CRITICAL** |
| **Class 5** | Capacity Deficiency | Division student count exceeds room physical seat capacity | **HIGH** |
| **Class 6** | Equipment Mismatch | Course requires lab apparatus missing in assigned room | **HIGH** |
| **Class 7** | Transit Collision | Consecutive classes in distant blocks with 0 transit time | **MEDIUM** |

---

## Five Standard Campus Failure Scenarios

1. **Scenario 1 (Master Outage):** 2-Day Maintenance Outage of Lab A-3 in Block A (CSE).
2. **Scenario 2 (Cross-Dept Faculty):** Shared Mathematics professor Dr. Sharma double-booked for CSE-1A and ECE-1B at 10:00 AM.
3. **Scenario 3 (Central Facility):** 1st-Year Central Workshop in Block C double-booked by CSE-1D and CIVIL-1B.
4. **Scenario 4 (Capacity Overflow):** 65-student division `MECH-3A` assigned to 40-seat tutorial room C105.
5. **Scenario 5 (Sudden Medical Leave):** ECE professor falls sick on Thursday morning, triggering substitute/recovery rerouting.

---

## Project Developer Skills (`.agents/skills/`)

| Skill | Focus |
|---|---|
| `fastapi-backend/SKILL.md` | Pydantic v2 validation, async sessions, and modular REST routers |
| `react-frontend/SKILL.md` | React, TypeScript, cascading selectors, and Tailwind CSS 2D timetable matrix |
| `dsa-orchestration/SKILL.md` | Conflict Graphs, BFS impact traversal, Priority Queue, Hash Maps, CSP Backtracking & Forward Checking |
| `os-multiprocessing/SKILL.md` | Multi-department parallel workers, priority queue with Aging, and lock ordering |
| `dbms-integrity/SKILL.md` | PostgreSQL normalization, ACID transactions, and Transactional Outbox |
