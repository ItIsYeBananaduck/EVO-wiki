---
type: audit-finding
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-219 — Map EVOfy Phase 2 → Core execution backbone

> Status: Draft mapping (execution-backbone alignment).
> Date: 2026-04-02.
> Parent: EVOC-217 — EVOfy → EVOconnect Mapping Layer.

## 1) Scope

This mapping anchors **EVOfy Phase 2 (Core task/method execution concerns)** to canonical **EVOconnect Core** execution-backbone issues.

The purpose is to ensure EVOfy remains implementation/migration work while Core remains authoritative for method model, execution pipeline, and task-binding semantics.

## 2) Canonical Core area (authoritative)

The following Core issues are the source of truth for this phase:

- **EVOC-190** — Connect Core Execution Backbone
- **EVOC-197** — Define Method domain model
- **EVOC-198** — Implement Method execution pipeline
- **EVOC-49** — Stabilize task execution state machine
- **EVOC-199** — Bind Tasks to Methods

## 3) EVOfy Phase 2 mapping

| EVOfy issue                                                                  | Relationship to Core               | Primary Core anchor(s)       | Delivery decision                                                                                                |
| ---------------------------------------------------------------------------- | ---------------------------------- | ---------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **EVOC-135** — Define ConnectTask schema                                     | **Implements**                     | EVOC-190, EVOC-49, EVOC-199  | **Remain** as implementation issue, with explicit Task → Method linkage semantics.                               |
| **EVOC-138** — Implement task lifecycle state machine                        | **Implements**                     | EVOC-49, EVOC-198            | **Remain** as implementation issue; lifecycle transitions must align with Core execution pipeline behavior.      |
| **EVOC-139** — Implement Task ↔ Action separation                           | **Partially overlaps**             | EVOC-197, EVOC-198, EVOC-199 | **Change** to align sequencing/terminology with Core method/task model and avoid introducing parallel semantics. |
| **EVOC-140** — Implement Talent reference binding to tasks                   | **Outdated by** Core binding model | EVOC-199, EVOC-197           | **Change** to Task → Method → Talent language or mark **deprecated** if left unconverted.                        |
| **EVOC-141** — Implement execution state tracking (waiting, running, review) | **Partially overlaps**             | EVOC-49, EVOC-198            | **Change** to align with Core state terminology and canonical execution-state definitions.                       |

## 4) Boundary rules for this mapping slice

1. **Core task binding wins**: Task → Method → Talent.
2. EVOfy issues in this phase may implement or migrate but must not redefine method, task, or talent authority boundaries.
3. EVOC-139 and EVOC-141 must be normalized to Core sequencing and naming.
4. EVOC-140 should be converted to Core binding language; if not converted, it should be explicitly marked **deprecated**.

## 5) Drift assessment

Drift risk is currently **medium** until all partial-overlap issues are normalized.

Primary risk triggers:

- EVOfy tickets that retain direct Task → Talent binding semantics.
- EVOfy lifecycle/state language that diverges from Core state machine and method-execution pipeline definitions.

Mitigation:

- Require Core anchor references in acceptance criteria for EVOC-139/140/141.
- Enforce terminology checks in issue grooming: Task → Method → Talent, and Core-aligned execution states.

## 6) Completion criteria for EVOC-219

This issue is complete when:

1. The EVOfy Phase 2 issues above are explicitly mapped to Core anchors.
2. Relationship type is stated for each mapping (`implements`, `partially overlaps`, `outdated by`).
3. A clear decision is present for each EVOfy issue (`remain` or `change`).
4. EVOC-140 has either been converted to Task → Method → Talent terminology or explicitly marked `deprecated`.