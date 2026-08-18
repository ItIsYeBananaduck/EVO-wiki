---
title: EVOtraining — Adapter Behavior
type: concept
tags: [evo, evotraining]
sources:
  - source-materials/mirrors/doctrine/EVOtraining — Adapter Behavior.md
updated: 2026-07-23
---
# EVOtraining — Adapter Behavior

> NOTE: This is a canonical doctrine note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

> RULE: All `related` entries must use Obsidian wiki link format.

---

## Purpose
Define how Alice learns and adapts workout behavior based on user performance, preferences, and recovery patterns.

---

## Core Principle
Training adapters learn from:

- physical performance (logs)
- interpreted effort and outcomes (journals)

They adapt:
- load
- reps
- sets
- exercise selection

---

## Definitions

**Workout Logs**  
Records of exercises, sets, reps, load, tempo, and completion.

**Performance Signals**  
Indicators such as:
- success/failure
- slowdown
- fatigue
- recovery

**Exercise Preference**  
Patterns where the user modifies or replaces exercises.

**Mesocycle**  
A structured training block used to establish patterns.

**Deload**  
A planned reduction in intensity or volume for recovery.

---

## System Structure

Training adaptation uses:

1. Logs → workout execution data  
2. Journals → interpretation of effort and performance  
3. Patterns → repeated behavior across sessions  
4. Adapter → applied changes  

---

## Rules

- No adaptation on single workouts  
- No adaptation during incomplete pattern formation  
- No adaptation during unresolved conflicts  
- User changes must be logged and interpreted before adaptation  

- Exercise changes must be treated as preference signals  
- Fatigue signals must be interpreted before load adjustment  

---

## Flow

Workout Execution → Logs → Journal Entry → Pattern Detection → Adapter Update  

Adaptation occurs only after validated patterns.

---

## Relationships

(Defined in frontmatter)

---

## Edge Cases / Special Handling

### Exercise Changes (User Overrides)

If the user replaces an exercise:

- log original exercise
- log replacement
- interpret as potential preference

If repeated:
→ becomes a preference pattern

---

### Performance Variation

If a workout is:

- slower
- harder
- incomplete

Journal must determine:
- off-day vs pattern

No adaptation on single instance.

---

### Deload Handling

Deload weeks must:

- be excluded from standard pattern detection
- be tagged as a separate pattern type

After multiple mesocycles:
→ deload becomes a recognized pattern

---

### Pattern Maturity

Patterns must progress:

1. First occurrence → observation  
2. Second occurrence → potential pattern  
3. Third+ occurrence → validated pattern  

Only validated patterns influence adaptation.

---

### Split Awareness (Critical)

Patterns must be evaluated per:

- Push
- Pull
- Legs

A pattern is not valid until:

- repeated within the same category

Cross-split patterns may exist but must be explicitly identified.

---

## Summary

EVOtraining adapters learn from:

- workout performance (logs)
- interpreted effort and outcomes (journals)

They adapt through:

- repeated, validated patterns
- mesocycle-level consistency

They are constrained by:

- user corrections
- proper pattern validation
- separation of training phases (e.g., deload)

---

Related notes: [[USER_LORA_TRAINING_IMPLEMENTATION]]

## Related
- [[EVO Architecture Bible]]
- [[EVOtraining — Coach Application Philosophy.md]]
- [[EVOtraining — Lab Supplement Intelligence.md]]
^[source-materials/mirrors/doctrine/EVOtraining — Adapter Behavior.md]
