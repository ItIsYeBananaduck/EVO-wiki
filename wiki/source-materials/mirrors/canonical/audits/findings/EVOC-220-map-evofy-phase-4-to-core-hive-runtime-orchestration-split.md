---
type: audit-finding
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-220 — Map EVOfy Phase 4 → Core Hive runtime + orchestration split

> Status: Draft mapping (Hive runtime/orchestration alignment).
> Date: 2026-04-02.
> Parent: EVOC-217 — EVOfy → EVOconnect Mapping Layer.

## 1) Scope

This mapping anchors **EVOfy Phase 4 (Hive integration work)** to canonical **EVOconnect Core** issues while explicitly splitting concerns between:

- **Hive runtime primitives** (node coordination/runtime mechanics), and
- **Connect orchestration behavior** (cross-device task orchestration semantics).

The purpose is to preserve Core authority and keep EVOfy Phase 4 implementation-only.

## 2) Canonical Core area (authoritative)

The following Core issues are the source of truth for this phase:

- **EVOC-189** — Hive Runtime — Node Coordination Layer
- **EVOC-172** — Connect v1 — Cross-Device Task Orchestration (Hive-Aware)

## 3) EVOfy Phase 4 mapping (runtime vs orchestration)

| EVOfy issue                                                        | Relationship to Core    | Runtime primitive anchor (Core) | Connect orchestration anchor (Core) | Delivery decision                                                                                                                 |
| ------------------------------------------------------------------ | ----------------------- | ------------------------------- | ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| **EVOC-147** — Integrate ConnectTask with Hive scheduling          | **Partially overlaps**  | EVOC-189                        | EVOC-172                             | **Change** to separate scheduler runtime mechanics from ConnectTask orchestration semantics; Core ordering decides tie-breakers. |
| **EVOC-146** — Implement node assignment for tasks                 | **Implements**          | EVOC-189                        | EVOC-172                             | **Remain** as implementation issue, but acceptance criteria must separate assignment primitive (runtime) from orchestration use. |
| **EVOC-149** — Implement async task execution via Hive             | **Partially overlaps**  | EVOC-189                        | EVOC-172                             | **Change** to distinguish runtime async execution behavior from orchestration-level task progression semantics.                   |
| **EVOC-150** — Implement task state synchronization across nodes   | **Implements**          | EVOC-189                        | EVOC-172                             | **Remain** as implementation issue with explicit split between node-state propagation and Connect-visible orchestration state.    |
| **EVOC-151** — Add fallback behavior for node failure              | **Implements**          | EVOC-189                        | EVOC-172                             | **Remain** as implementation issue while keeping runtime failover primitives and orchestration fallback policy distinct.          |

## 4) Boundary rules for this mapping slice

1. **Layer split is mandatory**: every issue in this slice must state runtime and orchestration scope separately.
2. **Core ordering decides semantics** for partial overlaps (especially EVOC-147 and EVOC-149).
3. EVOfy Phase 4 work is implementation/migration only and must not redefine Hive behavior.
4. Runtime primitive definitions follow EVOC-189; orchestration semantics follow EVOC-172.
5. Any acceptance criteria that merges both layers into one behavior statement should be rewritten before implementation proceeds.

## 5) Drift assessment

Drift risk is currently **medium-high** until partial-overlap issues are normalized to the two-layer split.

Primary risk triggers:

- EVOfy tickets that encode orchestration policy inside runtime assignment/scheduling logic.
- Async execution changes that alter Connect orchestration semantics without EVOC-172 alignment.
- Cross-node synchronization or fallback work that exposes runtime details as orchestration contract.

Mitigation:

- Require two explicit anchor references in each Phase 4 ticket: one runtime (EVOC-189), one orchestration (EVOC-172).
- Add review checklists that force separate acceptance criteria sections for runtime primitives vs orchestration behavior.
- Normalize EVOC-147 and EVOC-149 first, because they currently span both layers and can propagate semantic drift.

## 6) Completion criteria for EVOC-220

This issue is complete when:

1. The EVOfy Phase 4 issues above are explicitly mapped to both Core anchors.
2. Relationship type is stated for each mapping (`implements` or `partially overlaps`).
3. A clear decision is present for each EVOfy issue (`remain` or `change`).
4. EVOC-147 and EVOC-149 include explicit Core-ordering guidance for runtime vs orchestration semantics.
5. All mapped Phase 4 issues preserve implementation-only scope and avoid redefining Hive behavior in EVOfy.
