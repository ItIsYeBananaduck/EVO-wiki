---
title: EVOmind — Adapter Behavior
type: concept
tags: [evo, evomind]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# EVOmind — Adapter Behavior

#

> NOTE: This is a canonical doctrine note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

> RULE: All `related` entries must use Obsidian wiki link format.

---

## Purpose
Define how Alice learns emotional patterns, stress triggers, and behavioral responses in order to better support the user’s mental state.

---

## Core Principle
Mind adapters learn from:

- derived signals (from the signal model)
- contextual interpretation (journals)
- user-confirmed meaning (corrections)

Meaning is not inferred directly from signals.

---

## Definitions

**Derived Signals (Mind Logs)**  
Outputs from the signal model representing deviation from baseline.

**Interpretation**  
Alice’s explanation of what a signal likely means.

**Trigger**  
A context-specific cause of a mental or emotional state.

**Correction Event**  
User-provided clarification that overrides interpretation.

---

## System Structure

Signal Model  
→ Derived Signals (Mind Logs)  
→ Context + Comparison Layer  
→ Journal Interpretation  
→ Pattern Detection  
→ Adapter Update  

---

## Rules

- No adaptation on raw signals  
- No adaptation without interpretation  
- No adaptation without pattern confirmation  

- User correction overrides all interpretation  
- Context must be considered before meaning is assigned  
- Ambiguity must be preserved until resolved  

---

## Flow

Signal Detection  
→ Journal Interpretation  
→ (Optional) Conversational Journaling  
→ Pattern Detection  
→ Adapter Update  

---

## Relationships

(Defined in frontmatter)

---

## Edge Cases / Special Handling

### Misattributed Cause (Critical)

If:
- signals indicate stress in context A (e.g., work)

And user states:
- true cause is context B (e.g., pre-work interaction)

Then:
- journal must be corrected  
- previous interpretation downgraded  
- new pattern created based on corrected context  

---

### Signal Ambiguity (Comparison Layer)

If:
- signals could represent:
  - physical strain
  - emotional stress
  - cognitive load

Then:
- defer to comparison layer classification  
- avoid committing to a single interpretation  

---

### Silent Signals

If:
- deviation exists without clear context

Then:
- do not assign strong meaning  
- optionally initiate journaling  
- wait for repeated pattern  

---

### User Correction Loop

User correction must:

- override journal interpretation  
- be recorded as a correction event  
- influence future interpretation weighting  

---

### Conflicting Patterns

If:
- logs suggest pattern A  
- user confirms pattern B  

Then:
- prioritize user-confirmed meaning  
- retain signal data as supporting evidence  

---

### Pattern Evolution

Patterns must be tracked as:

1. Emerging → weak signal, low confidence  
2. Confirmed → repeated + corrected  
3. Evolving → shifting over time  

Adapters must:
- track transitions gradually  
- not abruptly replace patterns  

---

## Role of Conversational Journaling

Conversational journaling is:

- the primary mechanism for meaning clarification  
- the bridge between signal and truth  

It enables:

- trigger identification  
- context understanding  
- correction of interpretation  

---

## Pattern Requirements

A valid pattern must:

- repeat across time  
- align with derived signals  
- be confirmed or unchallenged by the user  

---

## Anti-Corruption Rules

Adapters must NOT train on:

- raw signals without interpretation  
- unresolved ambiguity  
- uncorrected misattributions  
- single-event emotional responses  

---

## Summary

EVOmind adapters learn from:

- derived signals (from the signal model)
- interpreted meaning (journals)
- user-confirmed truth (corrections)

They are governed by:

- context awareness  
- comparison layer disambiguation  
- cautious interpretation  

They evolve through:

- repeated, validated, and corrected **patterns**

## Related
- [[EVO Architecture Bible]]
^[wiki-native — no upstream source]
