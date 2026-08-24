---
title: Task Chain Definition
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Task Chain Definition.md
updated: 2026-07-24
---

# Task Chain Definition
## Purpose

Define Task Chains as preservable, callable, governed execution shortcuts within the EVO system.

Task Chains exist to let users preserve known multi-step work as reusable callable structures without losing visibility, control, or governance.

---

## Core Principle

> Recurring tasks repeat by schedule.

> Task Chains are preservable callable shortcuts.

A Task Chain is not simply a repeated task.

It is a **saved, callable, governed execution pattern**.

---

## Definition

A **Task Chain** is a preserved, repeatable, governed shortcut representing a string of tasks, Methods, and/or Talents working toward one goal.

A Task Chain may be:

- invoked by name
- invoked by phrase
- called directly in chat
- optionally scheduled as recurring

When invoked:

- Alice recognizes the Task Chain
- Alice adds the resulting work to the task list
- execution proceeds through normal governance

---

## What a Task Chain Contains

A Task Chain may contain:

- direct task steps
- Methods
- Talents
- nested Talents
- controlled Method variation where applicable

A Task Chain does not require every component to be a Talent.

This is why a Task Chain is broader than a Talent.

---

## Relationship to Tasks

### Task

A task is a user-facing unit of work.

### Recurring Task

A recurring task repeats on a schedule and may remain prompt-driven forever.

Recurring tasks are protected from cleanup only while they remain recurring.

If recurrence is removed:

- they return to normal task lifecycle rules
- they may eventually be deleted through cleanup

### Task Chain

A Task Chain is callable on demand and acts like a saved shortcut for a known execution pattern.

Unlike recurring tasks:

- Task Chains are explicitly preserved
- Task Chains remain available until explicitly deleted
- Task Chains are not removed through normal task cleanup

---

## Relationship to Methods and Talents

Task Chains are built from governed execution components.

### Methods

Methods provide structured execution for work that is not yet preserved as a Talent.

### Talents

Talents provide preserved, trusted repeatable execution patterns.

### Task Chains

Task Chains package tasks, Methods, and Talents into a callable preserved multi-step structure.

> A Task Chain is an invocation and preservation structure, not a separate governance class.

---

## Inference Compression Role

Task Chains compress inference further than Talents.

Talents compress trusted execution at the pattern level.

Task Chains compress trusted execution at the **multi-step task structure level**.

This means a Task Chain can reduce:

- repeated task decomposition
- repeated reasoning over known step order
- repeated reconstruction of known Methods and Talents
- repeated user prompting for known callable work

Task Chains improve speed and usability by preserving larger governed structures.

---

## Invocation Model

A Task Chain is triggered by a name or phrase.

Example model:

User says:

- “Run weekly cleanup”
- “Start client onboarding”
- “Do invoice closeout”

System behavior:

- Alice recognizes the Task Chain
- Alice adds the chain to the task list
- each step executes through normal governance rules

Invocation convenience does not change execution authority.

---

## Preservation Rule

Task Chains are **user-preserved artifacts**.

They are not automatically created just because a task is repeatable.

A Task Chain should be preserved only when:

- the work is likely to be reused
- the callable shortcut adds real value
- the structure is stable enough to remain understandable
- the chain remains bounded and governable

> Repeatability makes preservation possible.

> Future callable value makes preservation worthwhile.

Like Talent promotion, Task Chain preservation is explicit.

They remain available until the user explicitly deletes them.

---

## Governance Rule

Task Chains do not bypass governance.

Every component inside a Task Chain still obeys its normal rules:

- App Talent execution in closed domains remains governed internally by the application
- external or protected actions remain governed by Delegator
- Methods still require approval where applicable
- Talents still remain bounded by their preserved scope

> Callable does not mean unconstrained.

---

## Verification Rule

A Task Chain becomes **Verified** after 3 successful validated executions of the full preserved chain.

Because a Task Chain is locked and bounded, successful full-chain execution proves the contained Methods within that exact structure.

When a Task Chain becomes Verified:

- the Task Chain is trusted as a preserved callable execution structure
- contained non-Talent Methods are automatically promoted to Talents
- existing Talents remain unchanged
- Alice may offer to automate the Task Chain itself

Any variation must still validate independently through the Method path before being incorporated into a Talent.

---

## Approval and Acceleration

Task Chains may execute faster than ad hoc work because they may contain:

- preserved Talents
- known Methods
- validated nested execution patterns
- preserved multi-step structure

This can reduce:

- repeated reasoning
- repeated decomposition
- repeated approval burden on already trusted sub-patterns

However:

- non-Talent portions still follow Method rules
- protected actions still require appropriate governance
- the overall Task Chain must still succeed as a whole

---

## Boundedness Rule

A Task Chain must remain bounded by:

- its intended goal
- its component structure
- its approved execution surfaces
- its required governance rules

A Task Chain must not create:

- hidden authority transfer
- implicit scope expansion
- silent mutation of nested Talents or Methods
- execution outside the task manager’s visible structure

Task Chains can only be changed through explicit edits.

They cannot silently mutate their own structure at run time.

---

## Relationship to the Task Manager

Task Chains are not invisible automations.

They are visible, structured execution shortcuts that enter the task manager as work.

This means:

- the user can inspect them
- Alice can track them
- the system can govern them
- the resulting work remains part of the normal task history

Task Chains are preserved separately from ordinary task history.

This is why they survive cleanup even when old tasks, prompts, and Methods are removed.

---

## Recurring Execution

Task Chains may be scheduled for recurring execution.

Recurring execution:

- preserves the structure
- prevents deletion while active
- does not require promotion to Talent

Recurring Tasks and Task Chains are distinct:

- recurring tasks = scheduled execution
- task chains = callable preserved structures

---

## Final Principle

> A Task Chain is a preserved, callable shortcut for governed multi-step work.

It improves speed, recall, and usability without removing structure, visibility, or control.

---

## Related Notes

- [Talent Promotion Rule](https://www.notion.so/33ec72bad013814389d2efd20e39c2c6)
- [Talent Definition](https://www.notion.so/33ec72bad0138124922ee770d3aebbc0)
- [Talent Classes and Governance Boundaries](https://www.notion.so/344c72bad01381318dc4e44a02559619)
- [Execution Model: Intent → Effect → Execution](https://www.notion.so/343c72bad01381498ea5e9e5312270df)
- [Delegator Doctrine: Execution Authority](https://www.notion.so/343c72bad01381ef9ad0d496a384113b)
- [EVOconnect Talent Model](https://www.notion.so/33dc72bad0138188bcf7e7b995b3ac5f)
^[source-materials/mirrors/doctrine/Task Chain Definition.md]
