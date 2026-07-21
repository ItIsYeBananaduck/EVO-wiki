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
superseded_by:
  - source-materials/mirrors/doctrine/EVOconnect — Delegator & Governance Model.md
  - /Users/lsctech/Developer/EVO-wiki/wiki/components/alice.md
  - /Users/lsctech/Developer/EVO-wiki/wiki/doctrine/authority-governance-map.md
---

## Core Principle

> Alice orchestrates, evaluates, and learns from external agents to improve over time.

This file remains authoritative for Connect-specific orchestration, evaluation, and learning behavior. For shared Alice operational role, budget authority, and Delegator worker selection, see the canonical owner notes referenced below.

## Roles

Alice acts as:

- orchestrator  
- evaluator  
- learner  
- role assigner  

Orchestration is mediated by the Delegator. Alice assigns roles; the Delegator enforces authorization envelopes and routes workers.

## Agent Usage Model

Alice determines:

- which agent to use  
- what role it plays  
- how much access it gets  
- whether the output is useful  

Worker selection is delegated to the Delegator. Alice sets priorities, budget ceilings, and minimum-quality floors. The Delegator matches workers against capability contracts, authority envelopes, and cost/quality/latency tradeoffs within those bounds. Low cost never overrides safety, authority, or minimum-quality requirements.

## Cost Awareness

Alice learns:

- which agent is best for each task  
- when escalation is worth it  
- how to minimize usage  

Cost awareness belongs to the shared model-economics subsystem owned by `[[Governance & Authority Map]]` and exposed through `[[EVOconnect — Delegator & Governance Model]]`. Alice proposes usage optimizations; the Delegator operates within budget policies; the Cognitive Subsystem decides what persists.

## Learning Model

Alice learns from:

- task decomposition  
- corrections  
- accepted outcomes  
- review feedback  

Learning outcomes become proposals to the Cognitive Subsystem. Internalized strengths feed into [[EVOconnect — Method Specification Model]] as candidate Methods.

## Law

> Learning is based on accepted results, not raw outputs.

## Long-Term Effect

- reduced reliance on external agents  
- improved internal execution  
- stronger decision-making  

## 🔗 Relationships

Uses:
- [[EVOconnect — External Agent Governance Model]]

Before routing new work, consult:
- [[EVOconnect — Delegator & Governance Model]]
- [[Governance & Authority Map]]

For Alice operational role and budget authority, see first:
- /Users/lsctech/Developer/EVO-wiki/wiki/components/alice.md
- /Users/lsctech/Developer/EVO-wiki/wiki/doctrine/authority-governance-map.md

Guided By:
- [[EVOconnect — Delegator & Governance Model]]

Related:
- [[EVOconnect — Method Specification Model]]

## Final Principle

> External agents accelerate growth, but Alice internalizes their strengths.
