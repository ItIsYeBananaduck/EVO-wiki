---
title: Council_of_Echoes_Future_Spec
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Council_of_Echoes_Future_Spec.md
updated: 2026-07-24
---

# Council_of_Echoes_Future_Spec
EVOmind — Council of Echoes (Future Spec)
Purpose
Council of Echoes is a future reflective mode in EVOmind.

It allows users to consider multiple preserved perspectives at once.

It is designed for:
- deeper reflection
- perspective comparison
- understanding influence patterns
Core Principle
Alice remains the central intelligence.

Echoes are consulted, not merged.

Alice interprets and synthesizes.
Echoes provide bounded perspectives.
Non-Goals
Council of Echoes does NOT:
- merge identities
- create multi-agent debates
- replace Alice
- operate continuously in the background
Runtime Model
Base runtime:
Qwen base model
+ Alice stability layer
+ user adapter

Council mode:
Alice queries Echo instances individually
Each Echo runs in isolation
Responses are returned and synthesized
Consultation Flow
1. User query
2. Alice selects Echoes
3. Each Echo is queried independently
4. Responses are collected
5. Alice synthesizes final response
Echo Selection
Users can:
- select specific Echoes
- use predefined groups
- optionally consult all Echoes

Selection remains explicit, not automatic
Influence Learning
Alice can learn:
- which Echoes are consulted most
- which advice is followed
- domain-specific preferences

This creates an Echo Influence Profile
Echo Influence Profile
Tracks:
- echo_id
- usage frequency
- adoption frequency
- domain tags

This informs future suggestions but does not override user agency
Safety Boundaries
Alice must NOT:
- merge Echo tone into user identity
- silently bias all responses
- impersonate Echo sources

Echo influence must remain transparent and optional
UX Modes
1. Mediated Mode:
Alice summarizes Echo perspectives

2. Structured Mode:
Each Echo perspective is shown separately
Then synthesized
Future Implementation Notes
Hot-swapping adapters is possible at runtime,
but should be treated as an implementation detail.

Product model remains:
Alice consults Echoes, not becomes them.

## Related

^[source-materials/mirrors/doctrine/Council_of_Echoes_Future_Spec.md]
