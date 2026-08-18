---
title: EVO — Adapter Training System
type: concept
tags: [evo, system]
sources:
  - source-materials/mirrors/doctrine/EVO — Adapter Training System.md
updated: 2026-07-23
---
# EVO — Adapter Training System

> NOTE: This is a canonical doctrine note.
   All updates must preserve structure.
> Do not introduce conflicting definitions.

---

## Purpose
Define how Alice learns from user interactions using logs, journal entries, and living notes without corrupting training or misinterpreting user intent.

---

## Core Principle
Adapters are trained on:

- patterns in behavior (logs)
- interpreted meaning (journals)

Adapters are NOT trained on raw truth statements.

---

## Definitions

**Logs**  
Objective records of user actions, performance, and system events.

**Journal Entries**  
Alice’s interpretation of logs, including pattern detection and inferred meaning.

**Living Notes**  
User-confirmed knowledge. Represents truth, not training input.

**Adapter**  
A personalized model layer that adjusts behavior based on learned patterns.

---

## System Structure

The training system consists of three layers:

1. Evidence Layer → Logs  
2. Meaning Layer → Journal Entries  
3. Constraint Layer → Living Notes  

Training occurs between:
- Evidence + Meaning

And is bounded by:
- Constraint Layer

---

## Training Inputs

### Logs (Primary Signal)

Logs represent:
- actions
- performance
- decisions
- corrections

Logs are:
- objective
- time-based
- unfiltered

---

### Journal Entries (Meaning Layer)

Journal entries provide:
- interpretation of logs
- pattern recognition
- intent inference
- user corrections

Journals convert:
signal → meaning

---

### Living Notes (Constraint Layer)

Living Notes are NOT training data.

They act as:
- validation constraints
- interpretation boundaries
- truth overrides

---

## Training Model

Adapters are trained on:

(Log Patterns) + (Journal Meaning)

NOT:

(Living Notes)

---

## Pattern Requirements

A valid pattern must:
- repeat across relevant contexts
- appear in comparable scenarios
- demonstrate consistency

---

## Training Threshold

Adapters must NOT train on:
- single events
- incomplete patterns
- early observations

Initial training should occur after:
- full mesocycle completion (training domain)
OR
- sufficient repetition across contexts (other domains)

---

## Pattern Types

**Stable Pattern**
- consistent over time
- reinforced by multiple logs

**Emerging Pattern**
- repeated signals
- not yet stable

**Changing Pattern**
- previous pattern weakening
- new pattern forming

---

## Journal Role in Training

Journal entries must:
- describe observed patterns
- track repetition (first, second, third occurrence)
- propose meaning
- allow user correction

---

## User Correction Layer

User corrections:
- override journal interpretation
- influence training data
- prevent incorrect adaptation

---

## Conflict Handling in Training

If:
- logs suggest one pattern
- user states another

Then:
- user interpretation is prioritized
- logs are retained as evidence

---

## Deload Handling (Training-Specific)

Deload periods must:
- be excluded from standard pattern detection
- be tagged as a separate pattern class

After sufficient cycles:
- deload becomes a recognized pattern

---

## Domain-Specific Training

Each domain trains independently:

**Training**
- performance
- fatigue
- exercise preference

**Mind**
- mood
- triggers
- emotional patterns

**Learn**
- comprehension
- retention
- learning behavior

**Connect**
- workflows
- tools
- interaction patterns

---

## Adapter Update Strategy

Adapters update:
- incrementally
- based on validated patterns
- with bias toward recent correct data

---

## Anti-Corruption Rules

Adapters must NOT train on:
- unvalidated observations
- unresolved conflicts
- scratch data
- single-event signals

---

## Relationships

- [[EVO — Cognition Layer]]
- [[Alice Journal System]]
- [[Living Notes — Connect Knowledge System]]

---

## Summary

Adapters learn from:
- behavior (logs)
- meaning (journals)

They are constrained by:
- user-confirmed truth (living notes)

They evolve through:
- repeated, validated patterns

---

Related notes: [[EVOLoRA Mesh — Adapter Creation Pipeline]]

## Related
- [[EVO Architecture Bible]]
- [[EVO Wiki — One Alice, Many Rooms.md]]
- [[EVO — Cognition Layer.md]]
- [[EVO — Context Layer.md]]
- [[EVO — Cross-App Context Continuity.md]]
- [[EVO — Global Adapter Distribution Model.md]]
- [[EVO — Pane Pack Architecture.md]]
- [[EVO — Shared Embedding System.md]]
^[source-materials/mirrors/doctrine/EVO — Adapter Training System.md]
