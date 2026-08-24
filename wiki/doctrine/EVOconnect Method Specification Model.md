---
title: EVOconnect Method Specification Model
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVOconnect Method Specification Model.md
updated: 2026-07-24
---

# EVOconnect Method Specification Model
## Core Principle:

A Method is an exact, repeatable sequence of steps used to achieve a specific outcome.

### Definition:

- same goal
- same steps
- same expected outcome

Identity Rule:

If the steps change, it is a new Method.

If the outcome changes, it is a new Method.

A Method may have one controlled variation point. Repeated variation of the same step forms a variant. If a different step changes, it becomes a new Method.

- steps must be followed in order
- no step skipping
- no undefined behavior

### Lifecycle:

Method → (3 confirmed successful Alice executions) → Talent

## Method Requirement

All execution must be bound to a Method or a Talent.

If execution is not defined in a Method, it cannot proceed through the governed system.

This ensures:

- structured execution
- auditable behavior
- no uncontrolled action

### Success Rule:

A run only counts if the user confirms success. No feedback is inconclusive. Rejection is failure.

## Relationship to Task Chains

Methods may be used as components inside Task Chains.

A Task Chain does not replace Method definition.

It preserves and invokes higher-level structured work that may contain:

- tasks
- Methods
- Talents
- or a mixture of these

Within a Task Chain:

- a Method remains a Method unless promotion rules elevate it
- Method governance still applies
- Method constraints do not disappear because the Method is invoked inside a larger preserved structure
- Delegator enforcement still applies wherever the Method reaches external execution surfaces

Task Chains operate at a different layer than Methods:

- **Method** = explicit execution logic for a bounded pattern
- **Task Chain** = preserved callable structure that can organize multiple tasks, Methods, and Talents toward one goal

This means:

- Methods are the building blocks
- Task Chains are one way those building blocks can be preserved and invoked together

A verified Task Chain may contribute to Method promotion when its component Methods meet the relevant promotion rules, but the Task Chain itself does not redefine what a Method is.

## Connected To

- Tasks bind to Methods for execution
- Approved Methods may be stored in the Approved Method Library
- Methods may be promoted into Talents through the promotion path referenced by the Talent Model
- Methods may be used as components inside Task Chains, which preserve higher-level structured work rather than replacing Method structure

## Relationship to Task Chains

Methods may be used as components inside Task Chains.

A Task Chain does not replace Method definition.

It preserves and invokes higher-level structured work that may contain:

- tasks
- Methods
- Talents
- or a mixture of these

Within a Task Chain:

- a Method remains a Method unless promotion rules elevate it
- Method governance still applies
- Method constraints do not disappear because the Method is invoked inside a larger preserved structure
- Delegator enforcement still applies wherever the Method reaches external execution surfaces

Task Chains operate at a different layer than Methods:

- **Method** = explicit execution logic for a bounded pattern
- **Task Chain** = preserved callable structure that can organize multiple tasks, Methods, and Talents toward one goal

This means:

- Methods are the building blocks
- Task Chains are one way those building blocks can be preserved and invoked together

A verified Task Chain may contribute to Method promotion when its component Methods meet the relevant promotion rules, but the Task Chain itself does not redefine what a Method is.

## Connected To

- Tasks bind to Methods for execution
- Approved Methods may be stored in the Approved Method Library
- Methods may be promoted into Talents through the promotion path referenced by the Talent Model
- Methods may be used as components inside Task Chains, which preserve higher-level structured work rather than replacing Method structure

Relationships:

Enforced by Delegator & Governance Model. Feeds into Talent Model.

Final Principle:

Methods must be exact, repeatable, and strictly defined to prevent ambiguity and drift.

## Related

^[source-materials/mirrors/doctrine/EVOconnect Method Specification Model.md]
