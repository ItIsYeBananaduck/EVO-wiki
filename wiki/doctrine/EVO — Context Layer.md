---
title: EVO — Context Layer
type: concept
tags: [context, evo]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# EVO — Context Layer

> NOTE: This is a canonical doctrine note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

> RULE: All `related` entries must use Obsidian wiki link format.

---

## Purpose

Define how environmental, temporal, and semantic context is derived and used to influence signal interpretation and decision-making.

The context layer enables:

- interpretation of signals relative to environment  
- disambiguation of physical, emotional, and cognitive states  
- personalization based on user-specific meaning  

---

## Core Principle

Context is not location.

Context is the **meaning assigned to an environment by the user over time**.

---

## Definitions

**Location Data**  
Raw geographic coordinates or device-based positioning.

**Contextual Meaning**  
The interpreted significance of a location or environment for a specific user.

Examples:
- “Gym”
- “Work”
- “Home”
- “Travel”
- “Study”

---

**Context State**  
The current classification of the user’s environment.

---

## System Structure

Location / Time / Activity Signals  
→ Context Inference  
→ Context Classification  
→ Context Layer Output  
→ Used by Signal Model + Comparison Layer  

---

## Context Sources

Context may be derived from:

### Location
- repeated visits
- time spent
- user-labeled environments

---

### Temporal Patterns
- time of day
- day of week
- recurring schedules

---

### Behavioral Signals
- type of interaction
- typing patterns
- engagement level
- app usage

---

### Domain Activity
- active app (Training, Mind, Learn, Connect)
- workout session active
- learning session active

---

## Context Learning

Context is learned over time through:

- repeated exposure to locations  
- behavioral consistency within locations  
- user correction or labeling  

---

## Context Classification

Examples:

- Gym
- Work
- Home
- Study
- Unknown

Context classification is:

- probabilistic  
- user-specific  
- continuously updated  

---

## Context Influence

Context modifies interpretation of signals but does not override them.

Examples:

- High HR at Gym → likely physical strain  
- High HR at Work → likely emotional stress  
- High HR at Home late night → anomaly  

---

## Context + Signal Interaction

Context feeds into:

- Signal Model (baseline comparison)
- Comparison Layer (disambiguation)

It helps determine:

- physical vs emotional vs cognitive origin  
- expected vs abnormal behavior  

---

## Context + Baseline

Baseline must be:

- context-aware  
- maintained per environment  

Example:

- typing baseline at work ≠ typing baseline at home  
- HR baseline during workout ≠ HR baseline at rest  

---

## Context + Rerouting

Context may influence rerouting decisions:

- stress signals at work → suggest Mind  
- confusion during study → suggest Learn  
- performance signals → suggest Training  

---

## User Control

Users may:

- label environments  
- correct context classification  
- override inferred meaning  

User input takes priority over inferred context.

---

## Anti-Corruption Rules

The system must NOT:

- assume meaning from location alone  
- treat context as absolute truth  
- override signals with context  
- ignore user corrections  

---

## Summary

The Context Layer:

- transforms raw environment into user-specific meaning  
- influences signal interpretation  
- enables accurate disambiguation across domains  

It ensures:

- personalized understanding  
- correct attribution of signals  
- adaptive system behavior  

Context defines meaning, not just position.

---

Related notes: [[EVO — Cognition Layer]], [[Signal-Architecture]]

## Related
- [[EVO Architecture Bible]]
- [[EVO Wiki — One Alice, Many Rooms.md]]
- [[EVO — Adapter Training System.md]]
- [[EVO — Cognition Layer.md]]
- [[EVO — Cross-App Context Continuity.md]]
- [[EVO — Global Adapter Distribution Model.md]]
- [[EVO — Pane Pack Architecture.md]]
- [[EVO — Shared Embedding System.md]]
^[wiki-native — no upstream source]
