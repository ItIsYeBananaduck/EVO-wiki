---
title: Alice Cognitive Subsystem
type: concept
tags: [evo, alice, cognitive, journal, living-notes, knowledge-graph, preferences, governance, canonical]
sources:
  - /Users/lsctech/evo-cognitive-architecture-synthesis.md
  - /Users/lsctech/evo-cognitive-governance-spec.md
updated: 2026-07-21
---

# Alice Cognitive Subsystem

Alice's Cognitive Subsystem is the knowledge governance layer. It owns all canonical cognitive state and the governance logic that converts observations into canonical state.

## Ownership

The Cognitive Subsystem owns:

- Journal (canonical, append-only, multi-author)
- Living Notes (curated operational knowledge)
- Knowledge Graph (concept routing)
- Preferences (mutable, governance-committed)
- Understanding (derived working state)
- canonical cognitive proposals
- cognitive governance logic
- Memory Capsule generation

## Boundary rule

The Delegator does not own or govern this state. The Delegator may query Cognitive Subsystem indexes but cannot modify canonical state.

External harnesses and agents never directly mutate canonical cognitive state. They may return evidence, observations, candidate facts, suggested updates, or proposals. The Cognitive Subsystem decides what persists.

## Governance

All canonical state changes flow through the Cognitive Subsystem's governance authority:

- Direct user statements (Tier 1) → auto-promote after deterministic validation
- Alice-proposed updates (Tier 2) → review required for Preferences, KG, Living Notes
- System-derived updates (Tier 3) → auto-promote to Journal only

## Related

- [[Alice — Identity Layers]]
- [[Alice Capability Boundary]]
- [[Governance & Authority Map]]
- [[Hive Definition]]
