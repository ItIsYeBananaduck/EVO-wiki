---
title: Delegator Doctrine Execution Authority
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Delegator Doctrine Execution Authority.md"]
updated: 2026-07-24
---

# Delegator Doctrine Execution Authority
## Purpose

Define the role of Delegator as the enforcement layer for all external and protected execution.

---

## Core Principle

> Delegator is the sole authority on whether an action may be executed.

---

## Scope of Authority

Delegator governs all execution involving:

- external tools
- filesystem access
- bunker/protected paths
- cross-app interactions
- shared capability execution
- external agents (browser, terminal, APIs)

---

## Non-Scope

Delegator does not govern:

- intent formation
- effect modeling
- internal reasoning
- app-internal domain logic

---

## Execution Flow

Alice → Proposal → Delegator → Decision → Execution

---

## Decision Types

Delegator may:

- allow execution
- deny execution
- require approval
- require session/authentication

---

## Evaluation Criteria

Delegator evaluates:

- action type
- target identity
- target scope
- protection classification
- session state
- policy constraints

---

## Critical Rules

### 1. No Bypass

> No execution path may bypass Delegator for external or protected actions.

---

### 2. No Implicit Authority

- previous approval does not grant future authority
- talents do not grant permission
- patterns do not grant access

---

### 3. No Trust in Abstraction

> Delegator does not trust intent or inferred effect.

Only concrete execution is evaluated.

---

### 4. No Silent Expansion

- no widening of scope
- no chained escalation
- no implicit target expansion

---

### 5. Explicit Resolution Required

All actions must be resolved to:

- concrete operations
- explicit targets
- defined scope

before Delegator evaluation

---

## Relationship to Talents

Talents:

- define execution patterns
- improve efficiency and reuse
- reduce friction

Delegator:

- enforces permission
- validates scope
- prevents misuse

> Talents do not grant authority.

> Delegator does not assume trust.

---

## Relationship to Apps

- apps govern internal execution
- Delegator governs external execution

Connect is primarily Delegator-governed.

---

## Final Principle

> Alice proposes. Delegator enforces.

All execution authority flows through Delegator.

---

## Related Notes

- [Execution Model: Intent → Effect → Execution](https://www.notion.so/343c72bad01381498ea5e9e5312270df)
- Talent Classes and Governance Boundaries
- [Talent Definition](https://www.notion.so/33ec72bad0138124922ee770d3aebbc0)
- [Talent Promotion Rule](https://www.notion.so/33ec72bad013814389d2efd20e39c2c6)
- [EVOconnect Talent Model](https://www.notion.so/33dc72bad0138188bcf7e7b995b3ac5f)
- [Alice Delegation Governance Model](https://www.notion.so/341c72bad01381c7853fd79af12ca5eb)
- Talents as Inference Compression — EVOconnect
- [Data Sovereignty Doctrine](https://www.notion.so/33dc72bad01381eb9b4de2609f604c80)

---

Related notes: [[Alice Delegation Governance Model]]
