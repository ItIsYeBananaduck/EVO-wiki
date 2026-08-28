---
title: "EVOC-221 — Map EVOfy Phase 5 → Core plugin + capability system"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-04-02
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-221 — Map EVOfy Phase 5 → Core plugin + capability system

> Status: Draft mapping (plugin/capability alignment).
> Date: 2026-04-02.
> Parent: EVOC-217 — EVOfy → EVOconnect Mapping Layer.

## 1) Scope

This mapping anchors **EVOfy Phase 5 (plugin system work)** to canonical **EVOconnect Core** issues for:

- plugin contract and execution-surface boundaries,
- capability-based authorization behavior,
- and Talent participation in plugin-mediated execution.

The purpose is to keep EVOfy implementation/migration-bound while Core remains authoritative for plugin semantics, capability semantics, and Talent model semantics.

## 2) Canonical Core area (authoritative)

The following Core issues are the source of truth for this phase:

- **EVOC-195** — Connect Bounded Execution Surfaces
- **EVOC-191** — Talent Promotion and Automation System
- **EVOC-199** — Bind Tasks to Methods

## 3) EVOfy Phase 5 mapping

| EVOfy issue                                                              | Relationship to Core     | Primary Core anchor(s)       | Delivery decision                                                                                                                        |
| ------------------------------------------------------------------------ | ------------------------ | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **EVOC-152** — Define plugin interface (capabilities, permissions)       | **Partially overlaps**   | EVOC-195, EVOC-199, EVOC-191 | **Change** to explicitly inherit Core plugin/capability semantics; EVOfy must only express implementation contract details.             |
| **EVOC-153** — Implement plugin registration system                       | **Implements**           | EVOC-195                     | **Remain** as implementation issue, with acceptance criteria tied to Core-defined plugin lifecycle and bounded execution surfaces.       |
| **EVOC-154** — Implement capability-based access control                  | **Implements**           | EVOC-195, EVOC-191           | **Remain** as implementation issue, while capability authority and policy semantics remain Core-defined.                                 |
| **EVOC-155** — Implement plugin → Talent binding                          | **Partially overlaps**   | EVOC-191, EVOC-199, EVOC-195 | **Change** to avoid introducing alternate talent-binding semantics; Task/Method/Talent authority must follow Core ordering.             |
| **EVOC-156** — Implement secure execution sandbox for plugins             | **Implements**           | EVOC-195                     | **Remain** as implementation issue focused on sandbox mechanics that enforce Core boundaries without redefining policy or authority.     |

## 4) Boundary rules for this mapping slice

1. **Core defines plugin and capability semantics**; EVOfy defines implementation/migration mechanics only.
2. **Talent semantics are Core-owned**: EVOfy Phase 5 must not redefine Talent authority, promotion, or binding model.
3. **Task/Method binding semantics remain Core-owned** via EVOC-199 whenever plugin behavior intersects execution flow.
4. Partial-overlap issues (**EVOC-152**, **EVOC-155**) must be rewritten to reference Core anchors directly in acceptance criteria.
5. No EVOfy ticket in this slice may introduce competing plugin, capability, or talent terminology.

## 5) Drift assessment

Drift risk is currently **medium** because interface-definition and talent-binding work can easily introduce duplicate semantics.

Primary risk triggers:

- plugin interface docs that define capability meaning independently from Core;
- plugin → Talent binding logic that bypasses Core Task/Method/Talent ordering;
- sandbox or registration work that embeds policy semantics instead of enforcing Core policy.

Mitigation:

- Require explicit Core references (EVOC-195, EVOC-191, EVOC-199) in all Phase 5 acceptance criteria.
- Normalize EVOC-152 and EVOC-155 first, since both partially overlap Core semantics.
- Add review checks that separate "Core semantic anchor" from "EVOfy implementation mechanism" in each ticket.

## 6) Label and triage guidance

- Keep/add **`maps-to-core`** on this mapping issue and mapped EVOfy issues where useful.
- No additional label change is required at this time.

## 7) Completion criteria for EVOC-221

This issue is complete when:

1. The EVOfy Phase 5 issues above are explicitly mapped to Core anchors.
2. Relationship type is stated for each mapping (`implements` or `partially overlaps`).
3. A clear decision is present for each EVOfy issue (`remain` or `change`).
4. EVOC-152 and EVOC-155 include explicit Core-ordering guidance and remove competing semantics.
5. Mapped Phase 5 work remains implementation/migration-only and preserves Core authority for plugin/capability/talent definitions.