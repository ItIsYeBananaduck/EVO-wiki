---
tags:
  - concept/connect
  - concept/agents
  - concept/delegator
  - concept/security
  - concept/method
  - type/concept
status: active
source_of_truth: true
---

## Core Principle

> External agents do not obey Delegator internally.  
> They are constrained externally through controlled roles and execution paths.

---

## Enforcement Law

> Delegator cannot govern what it cannot intercept.

---

## Execution Paths

---

## 🅰️ Path A — Instruction-Based Execution (Preferred)

External agent provides instructions.

Alice:
- converts to Method  
- submits to Delegator  
- executes  

### Flow

External Agent → Instructions → Alice → Method → Delegator → Execution

### Benefits

- safest  
- fully auditable  
- Alice retains control  

---

## 🅱️ Path B — Agent-Defined Method (Restricted)

External agent provides full Method.

### Flow

External Agent → Method → Delegator → Execution

### Risk

- shifts execution authority  
- harder to control pivoting  

---

## Decision Rule

> Prefer Path A unless the Method is low-risk and well-understood.

---

## Roles

### Alice
- selects agent  
- defines role  
- evaluates output  
- learns  

### Delegator
- controls access  
- enforces constraints  

### External Agent
- provides reasoning  
- proposes solutions  

---

## 🔗 Relationships

Enforced By:
- [[EVOconnect — Delegator & Governance Model]]

Uses:
- [[EVOconnect — Method Specification Model]]

Related:
- [[EVOconnect — Multi-Agent Orchestration and Learning]]

---

## Final Principle

> External agents provide intelligence, but never authority.