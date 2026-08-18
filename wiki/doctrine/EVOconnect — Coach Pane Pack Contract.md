---
title: EVOconnect — Coach Pane Pack Contract
type: concept
tags: [connect, evo, pane]
sources:
  - source-materials/mirrors/doctrine/EVOconnect — Coach Pane Pack Contract.md
updated: 2026-07-23
---
# EVOconnect — Coach Pane Pack Contract

## Purpose

This note defines the architectural boundary for exposing Coach operational surfaces as portable Pane Packs inside EVOconnect.

This contract exists to prevent Coach from becoming a duplicated application runtime while still allowing Coach workflows to appear inside Connect.

---

## Core Philosophy

Coach remains an EVOtraining system.

The Pane Pack does not create:
- a second Coach runtime
- a second Alice runtime
- a second training system
- a second talent registry
- a second plan builder
- a second orchestration layer

The Pane Pack is a portable operational surface hosted by EVOconnect.

Coach logic continues to live inside EVOtraining.

---

## Pane Pack Purpose

The Coach Pane Pack exists to allow lightweight operational coaching workflows inside Connect without requiring the full standalone Coach application experience.

Examples:
- viewing a client workspace
- Alice Coach Chat
- quick roster access
- reviewing adherence or recovery trends
- reviewing workout progress
- lightweight plan interaction
- notifications and operational continuity

The Pane Pack should optimize for continuity, orchestration, and quick operational access.

The standalone Coach experience remains the full operational environment.

---

## Mobile Philosophy

Pane Packs may eventually support lightweight mobile operational experiences.

These lightweight experiences should:
- feel native to the platform
- remain fast and operationally focused
- avoid duplicating full desktop complexity
- prioritize continuity and quick interaction

This is similar to how EVOterminal or future lightweight operational apps may expose focused subsets of broader Connect functionality.

---

## Context Injection

When hosted inside Connect, the Coach Pane Pack may receive:
- active project context
- active client context
- active Hive context
- Alice conversational context
- pinned workspace context
- operational notifications

Context injection must not bypass Coach permission boundaries.

Coach delegation rules and talent boundaries still apply.

---

## Alice Behavior

Alice should behave consistently between:
- standalone Coach
- Coach Pane Pack inside Connect
- lightweight mobile operational surfaces

The hosting environment may change presentation and orchestration behavior, but not Coach authority boundaries.

---

## Ownership Boundary

EVOtraining owns:
- Coach business logic
- plan systems
- training structures
- adaptation systems
- coach/client relationships
- delegation rules
- coach analytics
- training cognition

EVOconnect owns:
- pane orchestration
- pane docking
- grouped pane composition
- workflow continuity
- cross-pane coordination
- workspace persistence
- operational overlays

---

## Deferred Architecture

The following remain intentionally unresolved for now:
- exact pane serialization format
- pane lifecycle synchronization rules
- offline pane persistence rules
- multi-device pane continuity
- standalone mobile pane-pack packaging
- pane-specific delegation overlays

These should remain doctrine-first until broader Pane Pack architecture stabilizes.

## Related
- [[EVOconnect — System Map]]
- [[EVO Architecture Bible]]
- [[EVOconnect — Action Bar & Mini Action Bar System.md]]
- [[EVOconnect — Connect Library & Unified Access Layer.md]]
- [[EVOconnect — Hive Node Architecture.md]]
- [[EVOconnect — Lightweight Talent Structure Addendum.md]]
- [[EVOconnect — Method Reconstruction Model.md]]
- [[EVOconnect — Mobile Operational Continuity.md]]
^[source-materials/mirrors/doctrine/EVOconnect — Coach Pane Pack Contract.md]
