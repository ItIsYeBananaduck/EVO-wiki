---
tags:
  - concept/connect
  - concept/method
  - concept/execution
  - concept/delegator
  - type/concept
status: active
source_of_truth: true
---

## Core Principle

> A Method defines how a task is executed in a structured, enforceable, and auditable way.

---

## Structure

Each Method contains ordered steps:

- action  
- target  
- constraints  
- expected outcome  

---

## Execution Rules

- no step skipping  
- no reordering  
- no undefined actions  

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

## 🔗 Relationships

Enforced By:
- [[EVOconnect — Delegator & Governance Model]]

Used By:
- [[EVOconnect — External Agent Governance Model]]

Related:
- [[EVOconnect — Multi-Agent Orchestration and Learning]]

---

## Final Principle

> Methods turn AI behavior into something enforceable and predictable.