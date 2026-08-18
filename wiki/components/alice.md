---
title: Alice — Canonical Component
type: concept
tags: [evo, alice, architecture, capability, safety, foreman, budget, multi-agent]
sources:
  - source-materials/mirrors/doctrine/Prompt Injection Boundary.md
  - source-materials/mirrors/doctrine/Alice-Limits.md
  - TODO: Add repo-relative source for evo-cognitive-architecture-synthesis.md
  - TODO: Add repo-relative source for evo-cognitive-governance-spec.md
updated: 2026-07-21
---

# Alice — Canonical Component

## Visibility Model
Alice operates within a strict context envelope. Teacher-visible outputs are limited to performance deltas unless explicitly expanded.

## Prompt Injection Defense
Web content is untrusted input. Alice cannot resolve secrets or privileged instructions from untrusted content.

## Authentication Boundary
Secret access requires authorization paths, tool grants, and explicit secret usage. None of these can be triggered by page content alone.

## Execution Trust
Capabilities are granted, not inferred. Even high-influence LoRAs cannot expand execution scope outside the Capability Map.

## Foreman
Alice is the persistent, primarily local foreman. She maintains the user relationship and conversational continuity, understands intent, performs lightweight local reasoning and appropriately bounded work, and decides when specialist execution is justified.

The model the user talks to does not have to be the model that performs the work.

Workers, analysts, librarians, medics, coding agents, and external harnesses receive narrowly scoped assignments with only the context necessary for their role.

## Budget Steward
Alice owns the user-facing budget relationship: token and spending preferences, budget ceilings, and priority tradeoffs. She assembles the minimum context required for delegation and presents validated results back to the user.

Cost optimization is permitted only after safety, authority, and minimum-quality requirements are satisfied.

## Related
- [[EVO Architecture Bible]]
- [[Governance & Authority Map]]
- [[Alice — Identity Layers]]
- [[EVOconnect — Multi-Agent Orchestration and Learning]]
- [[EVOconnect — Delegator & Governance Model]]

^[source-materials/mirrors/doctrine/Prompt Injection Boundary.md]
^[source-materials/mirrors/doctrine/Alice-Limits.md]
