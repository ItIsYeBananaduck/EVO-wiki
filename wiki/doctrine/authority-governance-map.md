---
title: Governance & Authority Map
type: concept
tags: [evo, governance, authority, delegation, escalation, method]
sources:
  - source-materials/mirrors/doctrine/ADR-001 - Dual Metric Learning Doctrine.md
  - source-materials/mirrors/doctrine/ADR-002 - Authority Separation Doctrine (Alice vs EVE).md
updated: 2026-07-20
---

# Governance & Authority Map

## Authority Separation
- EVO uses explicit authority separation rather than assumed capability.
- ADR-001 establishes dual-metric learning doctrine.
- ADR-002 defines Alice vs EVE authority boundaries.

## Delegator Enforcement
The runtime Delegator is the hard guardrail. It validates tool contracts, parameter schemas, and context allowlists.

## Method Approval
Methods require:
- explicit approval path
- bounded capability scope
- mandatory review before reuse
- governance override checks

## Escalation
Use escalation when:
- a concept exceeds one-boundary priority
- consent is missing for sensitive execution paths
- finality policy requires human confirmation

## Related
- [[EVO Architecture Bible]]
- [[Connect — Task Control Plane]]

^[source-materials/mirrors/doctrine/ADR-001 - Dual Metric Learning Doctrine.md]
^[source-materials/mirrors/doctrine/ADR-002 - Authority Separation Doctrine (Alice vs EVE).md]
