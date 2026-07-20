---
title: Hive Definition
type: concept
tags: [evo, hive, multi-device, lease, swarm, shared-state]
sources:
  - source-materials/mirrors/doctrine/Hive Definition.md
  - source-materials/mirrors/doctrine/Hive Shared State Backbone.md
updated: 2026-07-20
---

# Hive Definition

## What it is
The Hive is the collection of locally running Alice instances across a user’s devices.

## Invariants
- Shared chat, shared tasks, shared roster
- Exactly one lease holder at a time
- Lightweight UI, glyph-based status, not device-heavy
- Privacy-first: identity is pseudonymous and local

## Lease Holder
The active device holds the execution lease. It arbitrates swarm execution, canonical LoRA updates, and final Delegator validation.

## Swarm
Heavy compute can be distributed to idle hive members while the lease holder remains the single source of truth for user-visible execution state.

## Related
- [[Execution Lease Rule]]
- [[EVO Architecture Bible]]
- [[Connect — Task Control Plane]]

^[source-materials/mirrors/doctrine/Hive Definition.md]
^[source-materials/mirrors/doctrine/Hive Shared State Backbone.md]
