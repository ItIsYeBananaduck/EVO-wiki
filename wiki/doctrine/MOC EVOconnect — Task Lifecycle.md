---
title: MOC EVOconnect — Task Lifecycle
type: concept
tags: [connect, evo, lifecycle, moc]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# MOC EVOconnect — Task Lifecycle

[Task Lifecycle MOC](https://www.notion.so/33ec72bad01381b2a785ec94bb95ce7e)
Purpose
Maps the lifecycle of tasks from creation to execution under strict safety and authorization rules.
This MOC defines states, not behavior.

Core Guarantees
Tasks are non-actionable by default
Execution requires explicit authorization
Tool access is always gated

Lifecycle Stages
1) Created
Task exists
Method is defined
No tools available
Related: - Method Is Mandatory - No Tool Access During Planning

2) Planned
Method may be refined
Risks and tools identified
Still non-actionable
Related: - [[NOTION_PAGE:"[[Method Approval Path|Method Approval Path]]"]] - [[NOTION_PAGE:"[[Delegator Tool Hostage Rule|Delegator Tool Hostage Rule]]"]]

3) Awaiting Authorization
Task is blocked until one of the following is true: - Method approved by user - Method promoted to Talent
Related: - [[NOTION_PAGE:"[[Task Actionability Gate|Task Actionability Gate]]"]]

4) Authorized
Authorization granted via: - Method approval, or - Talent execution path
Scoped tool access is granted.
Related: - Scoped Tool Grants - Talent Execution Path

5) Executing
Task runs within scoped tool limits
No scope expansion allowed
Execution is monitored
Related: - [[NOTION_PAGE:"[[Delegator Tool Hostage Rule|Delegator Tool Hostage Rule]]"]]

6) Completed
Task finishes successfully
Outputs persisted
Method may be eligible for Talent promotion
Related: - [[NOTION_PAGE:"[[Talent Promotion Rule|Talent Promotion Rule]]"]]

7) Failed
Task fails safely
No partial side effects
User is informed if appropriate
Related: - Silent Failure Preference

Talent Lifecycle Overlay
Talents affect authorization, not task structure.
Promotion: [[NOTION_PAGE:"[[Talent Promotion Rule|Talent Promotion Rule]]"]]
Immutability: [[NOTION_PAGE:33ec72ba-d013-8198-9ea8-ca6805191903]]
Revocation: [[NOTION_PAGE:"[[Talent Revocation Rule|Talent Revocation Rule]]"]]

Architectural Rule
Tasks move forward only through explicit state transitions. There are no implicit promotions or executions.

## Related
- [[EVOconnect — System Map]]
- [[EVO Architecture Bible]]
- [[MOC EVOconnect — Agent System.md]]
- [[MOC EVOconnect — Cognitive & Execution Model.md]]
- [[MOC EVOconnect — Environment Integrations.md]]
- [[MOC EVOconnect — Escalation & Delegation.md]]
- [[MOC EVOconnect — Inference & Execution.md]]
- [[MOC EVOconnect — Methods & Talents.md]]
^[wiki-native — no upstream source]
