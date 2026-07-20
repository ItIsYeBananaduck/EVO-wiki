---
title: Alice Capability Boundary
type: concept
tags: [evo, alice, capability, visibility, safety, prompt-injection]
sources:
  - source-materials/mirrors/doctrine/Prompt Injection Boundary.md
  - source-materials/mirrors/doctrine/Alice-Limits.md
updated: 2026-07-20
---

# Alice Capability Boundary

## Visibility Model
Alice operates within a strict context envelope. Teacher-visible outputs are limited to performance deltas unless explicitly expanded.

## Prompt Injection Defense
Web content is untrusted input. Alice cannot resolve secrets or privileged instructions from untrusted content.

## Authentication Boundary
Secret access requires authorization paths, tool grants, and explicit secret usage. None of these can be triggered by page content alone.

## Execution Trust
Capabilities are granted, not inferred. Even high-influence LoRAs cannot expand execution scope outside the Capability Map.

## Related
- [[EVO Architecture Bible]]
- [[Governance & Authority Map]]

^[source-materials/mirrors/doctrine/Prompt Injection Boundary.md]
^[source-materials/mirrors/doctrine/Alice-Limits.md]
