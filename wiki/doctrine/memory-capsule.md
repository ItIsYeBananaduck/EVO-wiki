---
title: Memory Capsule
type: concept
tags: [evo, memory, capsule, context, application, device, cognitive]
sources:
  - EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVO — Safety Layer Architecture.md
  - EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOconnect External Agent Governance Model.md
updated: 2026-07-21
---

# Memory Capsule

A Memory Capsule is a low-latency, application-specific snapshot composed at session or interaction start.

A capsule is not a compressed copy of Alice's entire memory.

## Composition

Every capsule contains three layers:

1. Current application state
2. User's current state within that application
3. Relevant active knowledge required for the current interaction

## Properties

- Intentionally small and bounded by application scope
- Disposable — destroyed at session or application end
- Reproducible — regenerated from canonical state on demand
- Fast to generate — target <200ms for typical device
- Suitable for low-latency device use

## Application-bounded assembly

Capsule assembly queries only the indexes relevant to the current application and task. A calendar capsule never queries the Training database. A coding capsule does not load full Living Notes.

## Latency budgets

| Device | Application | Max size | Target assembly |
|--------|-------------|----------|-----------------|
| Watch | Calendar | 2,000 tokens | 100ms |
| Watch | Connect | 1,500 tokens | 80ms |
| Phone | Calendar | 4,000 tokens | 150ms |
| Phone | Connect | 3,000 tokens | 120ms |
| Phone | Living Notes | 3,000 tokens | 150ms |
| Desktop | Calendar | 8,000 tokens | 200ms |
| Desktop | Connect | 6,000 tokens | 180ms |
| Desktop | Living Notes | 12,000 tokens | 250ms |
| Desktop | Terminal | 16,000 tokens | 300ms |
| Desktop | Journal | 8,000 tokens | 200ms |
| Terminal | Journal | 16,000 tokens | 300ms |
| Terminal | Terminal | 32,000 tokens | 400ms |
| Cloud | Orchestration | 32,000 tokens | 400ms |
| Cloud | Governance | 32,000 tokens | 400ms |

## Fallback behavior

| Situation | Fallback |
|-----------|----------|
| Assembly fails | Minimal identity capsule (Soul + Role + device context) <200 tokens |
| Assembly exceeds budget | Stale capsule + staleness flag |
| Partial assembly failure | Partial capsule with `[MISSING: <query_type>]` markers |
| Quota/budget exceeded | Minimal identity capsule; escalate to user |

## Async enrichment

The initial capsule contains only immediate-interaction context. Enrichment requests additional knowledge after the session begins. Enrichment is announced and logged.

## Related

- [[Alice — Identity Layers]]
- [[Alice Cognitive Subsystem]]
- [[EVOconnect — System Map]]
^[EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVO — Safety Layer Architecture.md; EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOconnect External Agent Governance Model.md]
