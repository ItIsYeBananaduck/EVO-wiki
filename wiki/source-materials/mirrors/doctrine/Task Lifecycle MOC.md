# Task Lifecycle MOC

## Purpose
Maps the lifecycle of tasks from creation to execution under strict safety and authorization rules.

This MOC defines *states*, not behavior.

---

## Core Guarantees
- Tasks are non-actionable by default
- Execution requires explicit authorization
- Tool access is always gated

---

## Lifecycle Stages

### 1) Created
- Task exists
- Method is defined
- No tools available

Related:
- [[Method Is Mandatory]]
- [[No Tool Access During Planning]]

---

### 2) Planned
- Method may be refined
- Risks and tools identified
- Still non-actionable

Related:
- [[Method Approval Path]]
- [[Delegator Tool Hostage Rule]]

---

### 3) Awaiting Authorization
Task is blocked until one of the following is true:
- Method approved by user
- Method promoted to Talent

Related:
- [[Task Actionability Gate]]

---

### 4) Authorized
Authorization granted via:
- Method approval, or
- Talent execution path

Scoped tool access is granted.

Related:
- [[Scoped Tool Grants]]
- [[Talent Execution Path]]

---

### 5) Executing
- Task runs within scoped tool limits
- No scope expansion allowed
- Execution is monitored

Related:
- [[Delegator Tool Hostage Rule]]

---

### 6) Completed
- Task finishes successfully
- Outputs persisted
- Method may be eligible for Talent promotion

Related:
- [[Talent Promotion Rule]]

---

### 7) Failed
- Task fails safely
- No partial side effects
- User is informed if appropriate

Related:
- [[Silent Failure Preference]]

---

## Talent Lifecycle Overlay
Talents affect *authorization*, not task structure.

- Promotion: [[Talent Promotion Rule]]
- Immutability: [[Talent Immutability Rule]]
- Revocation: [[Talent Revocation Rule]]

---

## Architectural Rule
Tasks move forward only through explicit state transitions.
There are no implicit promotions or executions.