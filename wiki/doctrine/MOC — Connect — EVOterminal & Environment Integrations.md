---
title: MOC — Connect — EVOterminal & Environment Integrations
type: concept
tags: [evo, moc, terminal]
sources:
  - source-materials/mirrors/doctrine/MOC — Connect — EVOterminal & Environment Integrations.md
updated: 2026-07-23
---
# MOC — Connect — EVOterminal & Environment Integrations

Purpose
Define how Alice interacts with external tools and environments when direct provider integration is unavailable.
This layer enables Alice to: - Use local desktop tools - Interact with human-facing AI apps (Claude, etc.) - Execute governed automation - Maintain full auditability and safety

Core Idea
Not all intelligence is accessible via API or OAuth.
Instead of forcing provider integration, Connect supports: - Provider integrations (OAuth/API) - Environment integrations (EVOterminal) - Manual fallback workflows

Principles
Local-first execution
No fake integrations
Environment access is governed
No unrestricted automation
All actions must be auditable
User remains in control

Integration Types
1. Provider Integrations
OAuth providers
API providers
Local models
2. Environment Integrations
EVOterminal
Internal browser
Desktop applications
3. Fallback Integrations
Manual escalation packet export
Copy/paste workflows

Role of EVOterminal
EVOterminal is: - A governed execution surface- A bridge to local tools and apps- A controlled interface for Computer Alice- Not a raw terminal or unrestricted shell

Connected Systems
Connect - Hive Architecture
[Connect - Delegator & Governance](https://www.notion.so/33ec72bad013812bb1a2fcb216d082e3)
Connect - Task System
[Connect - Control Panel & Tools](https://www.notion.so/33ec72bad0138179bbe9fcbf3f4e0994)

Child Notes
[EVOterminal - Core Design](https://www.notion.so/33ec72bad01381fd8f0dda89110a1a14)
[Environment Integrations](https://www.notion.so/33ec72bad01381c3848bfeaa6c78a41d)
[Desktop Orchestration via Hive](https://www.notion.so/33ec72bad0138145bba1c17126c99f52)
Provider vs Environment Access #connect

## Related
- [[EVO Architecture Bible]]
- [[MOC EVOconnect — Agent System.md]]
- [[MOC EVOconnect — Cognitive & Execution Model.md]]
- [[MOC EVOconnect — Environment Integrations.md]]
- [[MOC EVOconnect — Escalation & Delegation.md]]
- [[MOC EVOconnect — Inference & Execution.md]]
- [[MOC EVOconnect — Methods & Talents.md]]
- [[MOC EVOconnect — Task Lifecycle.md]]
^[source-materials/mirrors/doctrine/MOC — Connect — EVOterminal & Environment Integrations.md]
