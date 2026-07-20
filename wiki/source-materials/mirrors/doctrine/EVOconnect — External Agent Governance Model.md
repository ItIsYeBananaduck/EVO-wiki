---
tags:
  - concept/connect
  - concept/agents
  - concept/security
  - concept/delegator
  - type/concept
status: active
source_of_truth: true
---


## Core Principle

> External agents provide suggestions, not authority.

---

## Enforcement Law

> Delegator cannot govern what it cannot intercept.

---

## Execution Paths

---

## Path A — Instruction-Based (Preferred)

External agent:

- provides instructions  

Alice:

- converts to Method  
- executes via Delegator  

---

## Path B — Method-Based (Restricted)

External agent:

- provides full Method  

Delegator:

- validates and executes  

---

## Rule

> Prefer Path A unless the task is low-risk and well understood.

---

## Learning Rule

Alice does NOT learn directly from external agents.

She learns only from:

- her own execution  
- user-approved outcomes  

---

## Role Definition

External agents are:

- advisors  
- specialists  
- method generators  

They are NOT:

- executors  
- trusted authorities  

---

## 🔗 Relationships

Uses:
- [[EVOconnect — Method Specification Model]]

Controlled By:
- [[EVOconnect — Delegator & Governance Model]]

---

## Final Principle

> External agents extend capability, but never control execution or learning directly.