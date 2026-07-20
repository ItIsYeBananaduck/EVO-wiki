---
tags:
  - concept/connect
  - concept/delegator
  - concept/governance
  - concept/safety
  - type/concept
status: active
source_of_truth: true
---


## Core Principle

> Delegator is the enforcement layer that ensures all execution is bounded, auditable, and controlled.

---

## Responsibilities

Delegator ensures:

- no unrestricted tool access  
- no silent execution  
- no uncontrolled mutation  
- all execution is Method-bound  
- all actions are auditable  

---

## Enforcement Rules

Alice cannot:

- access unrestricted terminal  
- execute arbitrary scripts  
- modify system state without approval  
- access external data without permission  

---

## Execution Law

> If it is not defined in a Method, it cannot be executed.

---

## Control Points

Delegator intercepts:

- tool access  
- method execution  
- external agent interaction  
- mutation attempts  

---

## 🔗 Relationships

Enforces:
- [[EVOconnect — Method Specification Model]]
- [[EVOconnect — External Agent Governance Model]]

Related:
- [[EVOconnect — Task Manager as Agent Supervision Layer]]

---

## Final Principle

> Delegator guarantees that Alice operates safely and predictably at all times.