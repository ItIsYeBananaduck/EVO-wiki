---
title: Connect — Task Control Plane
type: concept
tags: [evo, connect, tasks, governance, control-panel, hive]
sources:
  - source-materials/mirrors/doctrine/Connect - Task System.md
  - source-materials/mirrors/doctrine/Connect - Delegator & Governance.md
updated: 2026-07-20
---

# Connect — Task Control Plane

## Purpose
Connect governs task creation, execution, and supervision across EVO execution surfaces.

## Task Types
- Personal/manual tasks: reminders and direct actions, no AI execution
- AI tasks: delegated through the Delegator with method/talent binding

## Lifecycle
Created → Reviewed → Approved → Executed → Logged → Completed

## Control Surfaces
- Desktop orchestration via Hive
- Task manager as supervision layer
- Approval surfaces with configurable escalation paths

## Failure State Policy
Prefer silent failure with structured audit trails. Surface failures at control-boundary interfaces rather than deep in execution layers.

## Related
- [[EVOconnect — System Map]]
- [[Hive Definition]]
- [[Connect — Security & Privacy Model]]

^[source-materials/mirrors/doctrine/Connect - Task System.md]
^[source-materials/mirrors/doctrine/Connect - Delegator & Governance.md]
