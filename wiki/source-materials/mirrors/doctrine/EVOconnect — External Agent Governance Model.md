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

> External agents provide suggestions, not authority.

> External agents do not obey Delegator internally.
> They are constrained externally through controlled roles and execution paths.

---

## Enforcement Law

> Delegator cannot govern what it cannot intercept.

---

## Execution Paths

---

## 🅰️ Path A — Instruction-Based Execution (Preferred)

Also referred to as **Path A — Instruction-Based (Preferred)**.

External agent:

- provides instructions

Alice:

- converts to Method
- submits to Delegator
- executes via Delegator

### Flow

External Agent → Instructions → Alice → Method → Delegator → Execution

### Benefits

- safest
- fully auditable
- Alice retains control

---

## 🅱️ Path B — Agent-Defined Method (Restricted)

Also referred to as **Path B — Method-Based (Restricted)**.

External agent:

- provides full Method

Delegator:

- validates and executes

### Flow

External Agent → Method → Delegator → Execution

### Risk

- shifts execution authority
- harder to control pivoting

---

## Decision Rule

> Prefer Path A unless the Method is low-risk and well-understood.

> Prefer Path A unless the task is low-risk and well understood.

---

## Learning Rule

Alice does NOT learn directly from external agents.

She learns only from:

- her own execution
- user-approved outcomes

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

Enforced By:
- [[EVOconnect — Delegator & Governance Model]]

Controlled By:
- [[EVOconnect — Delegator & Governance Model]]

Uses:
- [[EVOconnect — Method Specification Model]]

Related:
- [[EVOconnect — Multi-Agent Orchestration and Learning]]

---

## Final Principle

> External agents provide intelligence, but never authority.

> External agents extend capability, but never control execution or learning directly.
