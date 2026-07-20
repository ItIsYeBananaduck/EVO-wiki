---
type: audit-finding
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-223 — Map EVOfy Phase 6 → Core governance enforcement layer

> Status: Draft mapping (governance-enforcement alignment).
> Date: 2026-04-02.
> Parent: EVOC-217 — EVOfy → EVOconnect Mapping Layer.

## 1) Scope

This mapping anchors **EVOfy Phase 6 (governance enforcement mechanics)** to canonical **EVOconnect Core** governance and execution-control issues.

The purpose is to keep EVOfy focused on implementation mechanics (checks, approvals, authorization validation, denial/fallback handling, and audit instrumentation) while Core remains authoritative for governance policy, control ordering, supervision semantics, and bounded execution behavior.

## 2) Canonical Core area (authoritative)

The following Core issues are the source of truth for this phase:

- **EVOC-190** — Connect Core Execution Backbone
- **EVOC-191** — Talent Promotion and Automation System
- **EVOC-194** — Connect Supervision and Learning Surface
- **EVOC-195** — Connect Bounded Execution Surfaces

## 3) EVOfy Phase 6 mapping

| EVOfy issue                                                            | Relationship to Core   | Primary Core anchor(s)       | Delivery decision                                                                                                                                      |
| ---------------------------------------------------------------------- | ---------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **EVOC-157** — Enforce capability checks before execution              | **Implements**         | EVOC-195, EVOC-190, EVOC-191 | **Remain** as implementation issue enforcing Core-defined capability and pre-execution constraints.                                                   |
| **EVOC-158** — Enforce user approval requirements                      | **Implements**         | EVOC-194, EVOC-195, EVOC-190 | **Remain** as implementation issue for approval gating and workflow instrumentation under Core supervision/control semantics.                         |
| **EVOC-159** — Implement audit logging for all Actions                | **Partially overlaps** | EVOC-194, EVOC-190           | **Change** to follow Core supervision and control ordering; logging must not introduce alternate governance semantics or alternate review states.      |
| **EVOC-160** — Implement Talent authorization validation               | **Implements**         | EVOC-191, EVOC-195, EVOC-190 | **Remain** as implementation issue for authorization enforcement tied to Core talent and bounded-execution authority model.                           |
| **EVOC-161** — Implement denial + fallback handling                    | **Partially overlaps** | EVOC-194, EVOC-195, EVOC-190 | **Change** so denial escalation, fallback sequencing, and operator override flow strictly inherit Core supervision/control ordering and surface rules. |

## 4) Boundary rules for this mapping slice

1. **Core owns governance semantics** (policy intent, control ordering, review model, supervision semantics).
2. EVOfy Phase 6 may implement enforcement mechanisms but must not redefine approval policy semantics, denial semantics, or operator authority.
3. **EVOC-159** and **EVOC-161** must explicitly reference Core supervision/control ordering in acceptance criteria and remove any parallel governance model language.
4. Audit logging in EVOfy is an execution transparency mechanism; it cannot become a separate governance-state model.
5. Denial and fallback behaviors must stay bounded by Core execution-surface constraints and Core-defined escalation pathways.

## 5) Drift assessment

Drift risk is currently **medium-high** because enforcement and observability work can easily become accidental policy-definition work.

Primary risk triggers:

- approval/denial logic defining new authority paths outside Core ordering;
- audit/event schemas introducing alternate governance states;
- fallback behavior bypassing Core supervision checkpoints;
- talent authorization checks not aligned with Core talent authority boundaries.

Mitigation:

- Require each Phase 6 EVOfy issue to name at least one Core anchor in acceptance criteria.
- Prioritize normalization of **EVOC-159** and **EVOC-161** before closing the phase slice.
- Add explicit review checklist language: "implements Core policy" vs "defines policy".

## 6) Label and triage guidance

- Keep/add **`maps-to-core`** on this mapping issue and mapped EVOfy issues where useful.
- No deprecation is recommended for this slice at this time; these issues remain implementation-relevant.

## 7) Completion criteria for EVOC-223

This issue is complete when:

1. The EVOfy Phase 6 issues above are explicitly mapped to Core anchors.
2. Relationship type is stated for each mapping (`implements` or `partially overlaps`).
3. A clear decision is present for each EVOfy issue (`remain` or `change`).
4. EVOC-159 and EVOC-161 explicitly inherit Core supervision/control ordering and remove any competing governance semantics.
5. The mapping makes explicit that Phase 6 stays implementation/migration-only while Core remains authoritative for governance behavior.
