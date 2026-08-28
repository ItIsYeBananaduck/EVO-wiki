---
title: "Evoconnect — Method Specification Model"
type: doctrine
tags: ['lsctech', 'doctrine', 'source-material', 'evo']
updated: 2026-08-24
---

## Core Principle

> A Method is an exact, repeatable sequence of steps used to achieve a specific outcome.

> A Method defines how a task is executed in a structured, enforceable, and auditable way.

---

## Definition

A Method requires:

- same goal
- same steps
- same expected outcome

---

## Structure

Each Method contains ordered steps:

- action
- target
- constraints
- expected outcome

---

## Identity Rule

> If the steps change, it is a new Method.

> If the outcome changes, it is a new Method.

---

## Variation Rule

A Method may have **one controlled variation point**:

- one step may vary
- repeated variation of the same step forms a variant

If a different step changes:

> it becomes a new Method

---

## Execution Rules

- steps must be followed in order
- no step skipping
- no reordering
- no undefined actions
- no undefined behavior

---

## Law

> If it is not in the Method, it does not happen.

---

## Delegator Integration

Delegator:

- validates each step
- enforces constraints
- can interrupt execution

---

## Observability

Each step records:

- input
- output
- status

---

## Lifecycle

Method → (3 confirmed successful Alice executions) → Talent

---

## Success Rule

A run only counts if:

- user confirms success

No feedback = inconclusive
Rejection = failure

---

## 🔗 Relationships

Enforced By:
- [[EVOconnect — Delegator & Governance Model]]

Used By:
- [[EVOconnect — External Agent Governance Model]]

Feeds Into:
- [[EVOconnect — Talent Model]]

Related:
- [[EVOconnect — Multi-Agent Orchestration and Learning]]

---

## Final Principle

> Methods must be exact, repeatable, and strictly defined to prevent ambiguity and drift.

> Methods turn AI behavior into something enforceable and predictable.