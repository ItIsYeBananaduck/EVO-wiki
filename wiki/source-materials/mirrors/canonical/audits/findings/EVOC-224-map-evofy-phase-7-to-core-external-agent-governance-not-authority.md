---
title: "EVOC-224 — Map EVOfy Phase 7 → Core external agent governance (NOT authority)"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-04-02
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-224 — Map EVOfy Phase 7 → Core external agent governance (NOT authority)

> Status: Draft mapping (external-agent governance alignment).
> Date: 2026-04-02.
> Parent: EVOC-217 — EVOfy → EVOconnect Mapping Layer.

## 1) Scope

This mapping anchors **EVOfy Phase 7 (swarm / multi-agent execution concerns)** to canonical **EVOconnect Core** external-agent governance issues.

The purpose is to preserve the Core rule that external agents are **suggestion systems, not authorities**. EVOfy Phase 7 remains implementation and migration work only: orchestration and aggregation logic can support escalation and candidate generation, but must not grant policy, execution, or approval authority to external-agent outputs.

## 2) Canonical Core area (authoritative)

The following Core issues are the source of truth for this phase:

- **EVOC-192** — External Agent Governance Pipeline
- **EVOC-52** — Detect escalation conditions in task execution
- **EVOC-53** — Build escalation packet for external reasoning
- **EVOC-55** — Resolve escalated multi-agent outputs into final action
- **EVOC-56** — Log successful escalation patterns for internalization
- **EVOC-203** — Convert external agent output into candidate Methods
- **EVOC-204** — Enforce external agent execution boundaries

## 3) EVOfy Phase 7 mapping

| EVOfy issue                                                  | Relationship to Core   | Primary Core anchor(s)  | Delivery decision                                                                                                                                                 |
| ------------------------------------------------------------ | ---------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **EVOC-162** — Implement multi-agent task distribution       | **Partially overlaps** | EVOC-192, EVOC-52, EVOC-204 | **Change** to explicitly route distribution through Core escalation detection/boundary controls and remove any implied autonomous authority model.              |
| **EVOC-163** — Implement coordination between agents         | **Partially overlaps** | EVOC-192, EVOC-53, EVOC-204 | **Change** so coordination semantics remain advisory and packet-driven under Core governance constraints, not independent execution authority.                 |
| **EVOC-164** — Implement task decomposition across agents    | **Partially overlaps** | EVOC-192, EVOC-52, EVOC-203 | **Change** to treat decomposition outputs as governed proposals/candidates only, with Core deciding admissibility and execution eligibility.                   |
| **EVOC-165** — Implement result aggregation                  | **Implements**         | EVOC-55, EVOC-203, EVOC-204 | **Remain** as direct implementation of Core-governed resolution where aggregated outputs become candidate Methods/proposals and never bypass Core gatekeeping. |
| **EVOC-167** — Implement conflict resolution between agents  | **Partially overlaps** | EVOC-55, EVOC-56, EVOC-204  | **Change** so conflict resolution produces auditable candidate outcomes and does not establish agent-level arbitration authority outside Core supervision.      |

## 4) Boundary rules for this mapping slice

1. **Core rule wins**: external agents are suggestion systems, not authorities.
2. Multi-agent outputs must always be represented as **candidates/proposals** under Core review and bounded-execution controls.
3. EVOfy Phase 7 must not introduce independent agent authority for approval, task ownership transfer, final-action selection, or policy interpretation.
4. Escalation detection, packet construction, resolution, and boundary enforcement semantics are Core-owned and inherited by EVOfy implementations.
5. Conflict and coordination logic in EVOfy can optimize signal quality and traceability, but cannot redefine adjudication authority.

## 5) Drift assessment

Drift risk is currently **high** for this slice because legacy EVOfy multi-agent language can imply autonomy/authority instead of governed advisory behavior.

Primary risk triggers:

- distribution or coordination paths that imply agent-to-agent delegated authority;
- decomposition outputs treated as executable plans without Core candidate conversion;
- aggregation pipelines selecting final action directly instead of proposing candidates;
- conflict resolution introducing non-Core arbitration layers or final-decision semantics.

Mitigation:

- Require all Phase 7 acceptance criteria to reference Core candidate-method and bounded-execution constraints.
- Prioritize normalization updates for **EVOC-162**, **EVOC-163**, **EVOC-164**, and **EVOC-167** to remove authority language.
- Keep **EVOC-165** as the canonical implementation pattern (candidate/proposal aggregation under governance).

## 6) Label and triage guidance

- Keep/add **`maps-to-core`** on this mapping issue and mapped EVOfy issues where useful.
- Keep/add **`governance-boundary`** where available to highlight authority-vs-suggestion checks.
- No deprecation is recommended for this slice at this time; issues remain implementation-relevant after wording/acceptance-criteria tightening.

## 7) Completion criteria for EVOC-224

This issue is complete when:

1. The EVOfy Phase 7 issues above are explicitly mapped to Core anchors.
2. Relationship type is stated for each mapping (`implements` or `partially overlaps`).
3. A clear decision is present for each EVOfy issue (`remain` or `change`).
4. EVOC-162/163/164/167 remove or avoid authority language and explicitly inherit Core external-agent governance boundaries.
5. EVOC-165 is preserved as a governed resolution implementation where outputs become candidate Methods/proposals.
6. The mapping clearly states that Phase 7 remains implementation/migration-only while Core stays authoritative for governance and final-action authority.