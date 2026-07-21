---
title: Governance & Authority Map
type: concept
tags: [evo, governance, authority, delegation, escalation, method]
sources:
  - source-materials/mirrors/doctrine/ADR-001 - Dual Metric Learning Doctrine.md
  - source-materials/mirrors/doctrine/ADR-002 - Authority Separation Doctrine (Alice vs EVE).md
updated: 2026-07-21
---

# Governance & Authority Map

## Authority Separation
- EVO uses explicit authority separation rather than assumed capability.
- Dual authority model: Delegator governs execution. Alice's Cognitive Subsystem governs canonical knowledge.
- ADR-001 establishes dual-metric learning doctrine.
- ADR-002 defines Alice vs EVE authority boundaries.

## Cognitive Subsystem Governance
Alice's Cognitive Subsystem owns canonical state governance. It is part of Alice's cognitive integrity system — not a second identity and not above Alice.

The Cognitive Subsystem receives proposals from agents and converts them to canonical state through authority tiers:
- Tier 1 (Direct User): Phil statements, auto-promote after deterministic validation
- Tier 2 (Alice-proposed): inferences and extracted knowledge, review required for Preferences, KG, Living Notes
- Tier 3 (System-derived): deterministic state changes, auto-promote to Journal only

## Delegator Enforcement
The Delegator governs execution only. It validates tool contracts, parameter schemas, context allowlists, output contracts, worker selection, authority envelopes, and budget enforcement while routing external work. The Delegator is not a governance authority for canonical knowledge.

## Budget Authority
Alice owns the user-facing budget relationship: token and spending preferences, budget ceilings, and priority tradeoffs. She sets the envelope outer bound. The Delegator enforces those policies during worker selection, capsule assembly, authority envelope construction, and task routing.

Cost optimization is permitted only after safety, authority, and minimum-quality requirements are satisfied.

## Method Approval
Methods require:
- explicit approval path
- bounded capability scope
- mandatory review before reuse
- governance override checks

## Proposal governance boundary
External agents and harnesses never directly mutate canonical cognitive state. They may return:
- evidence
- observations
- candidate facts
- suggested updates
- proposals

Harnesses and workers additionally return usage records: cost, quality, latency, retries, and outcome. These records inform future worker selection but do not by themselves promote or demote a worker.

Alice's Cognitive Subsystem decides what persists.

## Escalation
Use escalation when:
- a concept exceeds one-boundary priority
- consent is missing for sensitive execution paths
- finality policy requires human confirmation
- a Tier-2 proposal conflicts with higher-tier canonical state

## Related
- [[EVO Architecture Bible]]
- [[Alice Capability Boundary]]
- [[Alice — Identity Layers]]
- [[Hive Definition]]
- [[EVOconnect — System Map]]

^[source-materials/mirrors/doctrine/ADR-001 - Dual Metric Learning Doctrine.md]
^[source-materials/mirrors/doctrine/ADR-002 - Authority Separation Doctrine (Alice vs EVE).md]
