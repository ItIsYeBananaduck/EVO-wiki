---
title: "Delegator — Execution Governance Doctrine"
type: doctrine
tags: ['lsctech', 'doctrine', 'source-material', 'evo']
updated: 2026-04-25
---

## Purpose

Define the execution governance layer of the EVO ecosystem.

Delegator exists to ensure that all actions taken by Alice or any external system are:

- bounded
- observable
- auditable
- aligned with user authority
- consistent with system policy

---

## Core Principle

Intelligence proposes.  

Delegator governs.  

The system executes.

---

## Problem

Most AI systems collapse reasoning and execution into a single step:

- AI decides what to do
- AI directly uses tools
- AI executes actions

This results in:

- uncontrolled execution
- hidden behavior
- inconsistent outcomes
- difficulty enforcing safety
- no clear authority boundary

---

## Solution

Introduce a deterministic governance layer between reasoning and execution.

Delegator enforces:

- whether an action may occur
- how it may occur
- where it may occur
- under what conditions it may occur

---

## System Model

Alice → Proposal → Delegator → Decision → System → Tool → Result

---

## Key Separation

| Layer | Responsibility |
| --- | --- |
| Alice | reasoning, planning, proposing |
| Delegator | validation, policy enforcement |
| System | execution |
| Tools | capability surfaces |

---

## Non-Negotiable Constraint

> **No intelligence layer may directly execute actions.**

All execution must pass through Delegator.

---

## Proposal Model

All actions must be expressed as structured proposals.

A proposal defines:

- actor (user, Alice, external agent)
- domain (connect, training, mind, learn)
- action type
- execution surface
- provenance
- execution mode

---

## Delegator Responsibilities

Delegator does not decide *what to do*.

Delegator decides:

- Is this allowed?
- Does this require approval?
- Is this surface permitted?
- Is this action bounded?
- Is this safe to learn from?

---

## Decision Types

Delegator returns one of the following:

- allow
- allow_with_approval
- deny
- mediated_only
- candidate_only

---

## Execution Model

> Tools are not controlled by agents.

Tools are controlled by the system.

Agents do not use tools.

Agents request actions.

The system executes approved actions.

---

## Step-Level Governance

Execution is evaluated per step, not per task.

Every step must pass:

Proposal → Delegator → Decision → Execution

There is no persistent execution authority.

---

## Surface Model

Execution surfaces are controlled environments, not open tools.

### Examples

- Terminal → bounded command execution
- Browser → segmented interaction actions
- External agents → advisory only

---

## External Agent Rule

> External systems may propose.

They may not execute.

All external output:

- must pass through Alice
- must be validated by Delegator
- cannot directly trigger execution

---

## Learning Constraints

Delegator governs learning eligibility.

- Approved actions may contribute to Methods or Talents
- External outputs are not directly promotable
- Promotion requires validated execution

---

## Execution Constraints

The system must enforce:

- no persistent tool control
- no hidden loops
- no uncontrolled chaining
- no execution outside approved surfaces
- full auditability

---

## Authority Model

Final authority hierarchy:

1. User
2. Alice (mediated intelligence)
3. Delegator (enforcement layer)
4. System (execution layer)

Delegator enforces this hierarchy.

---

## Relationship to Alice

Delegator protects the integrity of Alice’s role.

Alice is:

- an extension of the user’s awareness, capability, and intent
- aligned before advising
- adaptive to the user
- bounded by trust and humanity

Delegator ensures that:

- Alice does not exceed user authority
- Alice remains aligned
- Alice cannot act outside governed boundaries

---

## Relationship to Data Sovereignty

Delegator supports the system-wide principle that user data remains under user control.

This is enforced by:

- preventing uncontrolled external execution
- controlling surface access
- restricting data exposure through actions
- ensuring all operations are intentional and bounded

---

## Why This Exists

Delegator is not a feature.

It is the layer that makes the system:

- safe
- consistent
- controllable
- scalable

Without Delegator:

- AI becomes execution authority
- behavior becomes unpredictable
- governance becomes impossible

---

## Principle

Delegator does not understand every action.

Delegator enforces the grammar of execution.

---

## End State

A mature system will:

- separate intelligence from execution completely
- evaluate every action deterministically
- maintain full user sovereignty
- allow powerful automation without loss of control

---

## Related Notes

- [Alice Identity Doctrine](https://www.notion.so/33dc72bad013811da04accd3f90303d3)
- [Data Sovereignty Doctrine](https://www.notion.so/33dc72bad01381eb9b4de2609f604c80)
- EVOconnect — External Agent Governance Model
- EVOconnect — Browser & Terminal Execution Model
- [EVOconnect — Execution Surface Selection & Approval Model](https://app.notion.com/p/EVOconnect-Execution-Surface-Selection-Approval-Model-33ec72bad01381729bcbf5daee689d40)
- EVOconnect — Method Specification Model
- Talents as Inference Compression — EVOconnect

---

## Summary

Do not allow intelligence to act directly.

Require it to propose.

Evaluate every proposal.

Execute only what is approved.

---

Related notes: [[Alice Delegation Governance Model]]