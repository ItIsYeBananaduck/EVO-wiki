---
title: "EVOC-222 — Map EVOfy Phase 3 → Core adapter / method model"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-04-02
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-222 — Map EVOfy Phase 3 → Core adapter / method model

> Status: Draft mapping (adapter/method-model alignment).
> Date: 2026-04-02.
> Parent: EVOC-217 — EVOfy → EVOconnect Mapping Layer.

## 1) Scope

This mapping anchors **EVOfy Phase 3 (OpenClaw adapter and action translation work)** to canonical **EVOconnect Core** method and execution-model issues.

The purpose is to keep EVOfy adapter work implementation/migration-bound while Core remains authoritative for method semantics, action semantics, and task-binding behavior.

## 2) Canonical Core area (authoritative)

The following Core issues are the source of truth for this phase:

- **EVOC-190** — Connect Core Execution Backbone
- **EVOC-197** — Define Method domain model
- **EVOC-198** — Implement Method execution pipeline
- **EVOC-199** — Bind Tasks to Methods
- **EVOC-195** — Connect Bounded Execution Surfaces

## 3) EVOfy Phase 3 mapping

| EVOfy issue                                                                    | Relationship to Core                  | Primary Core anchor(s)       | Delivery decision                                                                                                  |
| ------------------------------------------------------------------------------ | ------------------------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **EVOC-142** (deprecated) — Wrap OpenClaw execution loop behind Delegator     | **Migration-only support for**        | EVOC-190, EVOC-195           | **Change** to include/retain explicit `migration-only` labeling and preserve Core execution-boundary semantics.   |
| **EVOC-144** — Create adapter for tool invocation → Action model              | **Migration-only support for**        | EVOC-197, EVOC-198, EVOC-199 | **Change** to include/retain explicit `migration-only` labeling and enforce Core-defined Action/Method semantics. |
| **EVOC-143** — Replace direct tool calls with governed Actions                | **Implements**                        | EVOC-197, EVOC-198, EVOC-195 | **Remain** as implementation issue translating legacy behavior into Core-governed Actions.                        |
| **EVOC-145** — Remove legacy agent loop behavior                              | **Migration-only support for**        | EVOC-190, EVOC-198, EVOC-195 | **Change** to include/retain explicit `migration-only` labeling and keep scope limited to legacy removal.         |
| **EVOC-148** — Normalize tool outputs into EVO format                         | **Migration-only support for**        | EVOC-197, EVOC-198, EVOC-199 | **Change** to include/retain explicit `migration-only` labeling and avoid introducing parallel output semantics.  |

## 4) Boundary rules for this mapping slice

1. **Core ordering and semantics win** for method, action, and task-binding behavior.
2. EVOfy adapter issues may translate, wrap, and normalize legacy OpenClaw behavior but must not define new method/action/task semantics.
3. EVOC-142, EVOC-144, EVOC-145, and EVOC-148 should consistently carry **migration-only** labeling.
4. EVOC-143 should remain focused on governed Action adoption and explicitly reference Core action/method semantics.

## 5) Drift assessment

Drift risk is currently **high** if adapter work begins redefining action or method semantics rather than translating legacy behavior.

Primary risk triggers:

- EVOfy adapter tickets introducing Action semantics not anchored to EVOC-197/EVOC-198.
- Legacy-loop removal work that silently changes execution control behavior outside EVOC-190/EVOC-195 boundaries.
- Output-normalization changes that create non-Core schemas without Task → Method alignment.

Mitigation:

- Require Core issue references in acceptance criteria for all Phase 3 EVOfy tickets.
- Enforce `migration-only` labeling for EVOC-142/144/145/148 during grooming.
- Add explicit review check that adapter code translates legacy behavior without redefining Core concepts.

## 6) Completion criteria for EVOC-222

This issue is complete when:

1. The EVOfy Phase 3 issues above are explicitly mapped to Core anchors.
2. Relationship type is stated for each mapping (`implements` or `migration-only support for`).
3. A clear decision is present for each EVOfy issue (`remain` or `change`).
4. EVOC-142/144/145/148 are explicitly treated as `migration-only` work.
5. EVOC-143 is explicitly scoped as implementation of Core-governed Action semantics, not a semantics-defining issue.