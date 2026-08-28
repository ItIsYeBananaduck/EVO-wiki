---
title: "EVOC-225 — Map EVOfy Phase 8 → Core surface integration"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-04-02
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-225 — Map EVOfy Phase 8 → Core surface integration

> Status: Draft mapping (surface-integration alignment).
> Date: 2026-04-02.
> Parent: EVOC-217 — EVOfy → EVOconnect Mapping Layer.

## 1) Scope

This mapping anchors **EVOfy Phase 8 (Connect surface integration work)** to canonical **EVOconnect Core** interaction, supervision, and multi-surface experience issues.

The purpose is to keep EVOfy Phase 8 implementation-only while Core remains authoritative for interaction contracts, execution-flow ordering, supervision semantics, and cross-surface UX guarantees.

## 2) Canonical Core area (authoritative)

The following Core issues are the source of truth for this phase:

- **EVOC-210** — Connect Core Interaction Contract
- **EVOC-211** — Define Method proposal interaction contract
- **EVOC-213** — Define task execution visibility contract
- **EVOC-214** — Define execution control contract
- **EVOC-215** — Define timeline visualization and density rules
- **EVOC-212** — Define task identity and ownership indicators
- **EVOC-216** — Define execution transparency contract
- **EVOC-194** — Connect Supervision and Learning Surface
- **EVOC-196** — Connect Multi-Surface UX and Onboarding
- **EVOC-50** — Wire Alice chat into task execution flow

## 3) EVOfy Phase 8 mapping

| EVOfy issue                                                         | Relationship to Core   | Primary Core anchor(s)          | Delivery decision                                                                                                                                               |
| ------------------------------------------------------------------- | ---------------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **EVOC-168** — Bind ConnectTask to UI task cards                   | **Implements**         | EVOC-213, EVOC-212, EVOC-216    | **Remain** as direct surface implementation of Core task execution visibility, ownership/identity, and transparency rules.                                    |
| **EVOC-166** — Integrate chat as task context layer                | **Partially overlaps** | EVOC-50, EVOC-210, EVOC-214     | **Change** so chat is explicitly positioned as a Core-defined execution-flow surface and not treated as an independent orchestration layer.                   |
| **EVOC-169** — Implement approval UI for Actions                   | **Implements**         | EVOC-211, EVOC-214, EVOC-216    | **Remain** as direct implementation of Core proposal/approval and execution-control interactions with explicit auditability and status signaling.              |
| **EVOC-170** — Implement task progress visualization               | **Implements**         | EVOC-213, EVOC-215, EVOC-216    | **Remain** as direct implementation of Core timeline-density, execution-visibility, and transparency behavior.                                                |
| **EVOC-171** — Implement notification-driven task updates          | **Implements**         | EVOC-213, EVOC-194, EVOC-196    | **Remain** as direct implementation of Core supervision feedback loops and multi-surface update consistency.                                                  |

## 4) Boundary rules for this mapping slice

1. **Core ordering wins**: chat/task execution flow is defined by Core and must be implemented in EVOfy surfaces without reordering semantics.
2. EVOfy Phase 8 may implement visual/UI behavior, event wiring, and notification pathways, but must not redefine interaction contract semantics.
3. Approval, control, visibility, and transparency states must inherit Core-defined contract terms and transitions.
4. Surface integrations must preserve task identity and ownership indicators defined by Core across all touchpoints.
5. EVOfy Phase 8 remains implementation/migration-only; Core remains authoritative for supervision behavior and multi-surface UX contract semantics.

## 5) Drift assessment

Drift risk is currently **moderate** for this slice, concentrated in chat-context and execution-flow ordering language.

Primary risk triggers:

- chat integration patterns that imply an alternate execution backbone outside Core ordering;
- surface status labels or transitions diverging from Core interaction/transparency contracts;
- progress/timeline rendering that ignores Core density and overlap rules;
- notification updates that diverge by surface and weaken supervision consistency.

Mitigation:

- Normalize **EVOC-166** language to explicitly inherit Core execution-flow and chat-binding semantics from EVOC-50 and EVOC-210.
- Require each mapped EVOfy issue to cite at least one Core anchor in acceptance criteria.
- Add contract review checks for state names/transitions used by UI cards, approvals, timeline views, and notifications.

## 6) Label and triage guidance

- Keep/add **`maps-to-core`** on this mapping issue and mapped EVOfy issues where useful.
- Keep/add **`surface-contract`** where available to highlight interaction and UX contract conformance.
- No deprecation is recommended for this slice at this time; mapped issues remain implementation-relevant after the EVOC-166 scope correction.

## 7) Completion criteria for EVOC-225

This issue is complete when:

1. The EVOfy Phase 8 issues above are explicitly mapped to Core anchors.
2. Relationship type is stated for each mapping (`implements` or `partially overlaps`).
3. A clear decision is present for each EVOfy issue (`remain` or `change`).
4. EVOC-166 is explicitly constrained to surface-level implementation of Core execution-flow behavior and does not imply independent orchestration semantics.
5. EVOC-168/169/170/171 are preserved as direct surface implementations of Core interaction, supervision, and multi-surface UX contracts.
6. The mapping clearly states that Phase 8 remains implementation/migration-only while Core stays authoritative for contract semantics and flow ordering.