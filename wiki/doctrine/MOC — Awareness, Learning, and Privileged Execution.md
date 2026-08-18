---
title: MOC — Awareness, Learning, and Privileged Execution
type: concept
tags: [execution, moc]
sources:
  - source-materials/mirrors/doctrine/MOC — Awareness, Learning, and Privileged Execution.md
updated: 2026-07-23
---
# MOC — Awareness, Learning, and Privileged Execution

Parent
MOC - EVOconnect (Modular OS Layer) ## Purpose Define how Alice:
remains present across all surfaces
respects strict learning boundaries
safely executes privileged (sudo-level) actions
integrates with Delegator, Vault, and Task System
This MOC unifies: - Awareness vs Learning - Training Mode - Privileged Execution - Vault-based credential access - Recurring governed automation

Core Concepts
[Awareness vs Learning Boundary](https://www.notion.so/33dc72bad0138103a116cd8a26f7ba50)
EVOconnect — Talent Training Mode (Intentional Learning) 1
[Privileged Execution Model](https://www.notion.so/33dc72bad0138165a5aac2af4808ef05)
[Vault-Based Credential Access](https://www.notion.so/33ec72bad013810b8476f725a8e8eeab)
[Recurring Talents & Scheduled Execution](https://www.notion.so/33ec72bad013817da3adc52fa01d3dcd)

System Relationships
Governance Layer
[Connect - Delegator & Governance](https://www.notion.so/33ec72bad013812bb1a2fcb216d082e3)
Execution Layer
[EVOterminal - Core Design](https://www.notion.so/33ec72bad01381fd8f0dda89110a1a14)
[Provider vs Environment Access](https://www.notion.so/33ec72bad01381a29ecbf54ba4aa03fc)
Task System
Connect - Task System
Security Layer
Connect - Security & Privacy Model

Key Principles
Alice is always present, but never invasive
Learning requires explicit user intent
Privileged actions require explicit approval
Credentials are never exposed, only used via vault
All execution is governed, logged, and auditable

Design Boundaries
Awareness
Always active
No learning
No memory creation
Learning
Training Mode only
Method creation required
User approval required
Execution
Must bind to Method
Must pass Delegator
Must be logged
Privileged Execution
Requires explicit approval
Vault-mediated
No silent escalation

Example Flow
System Cleanup Talent
User enters Training Mode
Performs cleanup via EVOterminal
Alice constructs Method
User approves Talent
User schedules weekly execution
Future runs:
prompt for sudo approval (unless whitelisted)
execute via EVOterminal
log results

Future Extensions
Trust tiers for privileged Talents
Adaptive approval suggestions
Storage-aware system monitoring
Auto-suggested cleanup (non-learning awareness)

Core Takeaway
Awareness gives Alice presence.Training Mode gives Alice growth.Delegator ensures Alice remains safe.Vault ensures Alice never becomes dangerous.
#connect #cognition

## Related
- [[EVO Architecture Bible]]
- [[MOC EVOconnect — Agent System.md]]
- [[MOC EVOconnect — Cognitive & Execution Model.md]]
- [[MOC EVOconnect — Environment Integrations.md]]
- [[MOC EVOconnect — Escalation & Delegation.md]]
- [[MOC EVOconnect — Inference & Execution.md]]
- [[MOC EVOconnect — Methods & Talents.md]]
- [[MOC EVOconnect — Task Lifecycle.md]]
^[source-materials/mirrors/doctrine/MOC — Awareness, Learning, and Privileged Execution.md]
