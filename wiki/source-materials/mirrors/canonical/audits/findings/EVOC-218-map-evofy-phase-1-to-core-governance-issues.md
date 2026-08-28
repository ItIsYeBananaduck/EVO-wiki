---
title: "EVOC-218 — Map EVOfy Phase 1 → Core governance issues"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-04-02
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-218 — Map EVOfy Phase 1 → Core governance issues

> Status: Draft mapping (governance alignment).
> Date: 2026-04-02.
> Parent: EVOC-217 — EVOfy → EVOconnect Mapping Layer.

## 1) Scope

This mapping anchors **EVOfy Phase 1 (Safety & Governance Enforcement)** to canonical **EVOconnect Core** governance/execution issues.

The purpose is to keep EVOfy implementation-bound and prevent concept drift by treating Core semantics as authoritative for execution-boundary decisions.

## 2) Canonical Core area (authoritative)

The following Core issues are the source of truth for this phase:

- **EVOC-190** — Connect Core Execution Backbone
- **EVOC-49** — Stabilize task execution state machine
- **EVOC-214** — Define execution control contract
- **EVOC-195** — Connect Bounded Execution Surfaces

## 3) EVOfy Phase 1 mapping

| EVOfy issue                                                            | Relationship to Core                                    | Primary Core anchor(s)       | Delivery decision                                                                                |
| ---------------------------------------------------------------------- | ------------------------------------------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------ |
| **EVOC-132** — Implement Delegator enforcement boundary                | **Implements**                                          | EVOC-190, EVOC-195           | **Remain** as implementation issue with explicit references to Core boundary semantics.          |
| **EVOC-136** — Enforce bounded execution model (no autonomous loops)   | **Implements**                                          | EVOC-49, EVOC-195            | **Remain** as implementation issue; no independent model semantics allowed.                      |
| **EVOC-134** — Remove direct execution pathways from OpenClaw runtime  | **Migration-only support for** Core governance boundary | EVOC-190, EVOC-214, EVOC-195 | **Change** to include/retain `migration-only` labeling and avoid net-new governance definitions. |
| **EVOC-133** — Implement Actionability Gate (pre-execution validation) | **Implements**                                          | EVOC-49, EVOC-214            | **Remain** as implementation issue tied to Core validation and control semantics.                |
| **EVOC-137** — Add execution permission checks                         | **Implements**                                          | EVOC-214, EVOC-195           | **Remain** as implementation issue; authority model must remain Core-defined.                    |

## 4) Boundary rules for this mapping slice

1. **Core ordering and Core semantics win** for all execution-boundary and governance decisions.
2. EVOfy tickets in this slice may implement, enforce, or migrate—but must not redefine method/task/control models.
3. Any EVOfy language that appears to introduce new governance authority should be rewritten to reference Core anchors.
4. EVOC-134 should be treated primarily as legacy pathway removal and consistently tagged as **migration-only**.

## 5) Drift assessment

Drift risk is currently **low** if EVOfy remains constrained to enforcement of boundaries already defined in Core.

Primary risk trigger:

- EVOfy issues introducing standalone semantics (state/control/authority) without explicit linkage to EVOC-190 / EVOC-49 / EVOC-214 / EVOC-195.

Mitigation:

- Require Core cross-references in implementation acceptance criteria for all mapped EVOfy issues in this phase.

## 6) Completion criteria for EVOC-218

This issue is complete when:

1. The EVOfy Phase 1 issues above are explicitly mapped to Core anchors.
2. Relationship type is stated for each mapping (`implements` or `migration-only support for`).
3. A clear decision is present for each EVOfy issue (`remain` or `change`).
4. EVOC-134 is clearly marked as migration-only work.