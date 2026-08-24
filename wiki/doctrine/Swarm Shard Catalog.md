---
title: Swarm Shard Catalog
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Swarm Shard Catalog.md
updated: 2026-07-24
---

# Swarm Shard Catalog
[Swarm Shard Catalog](https://www.notion.so/33ec72bad01381e9a2a8d6c0a0902c84)
Purpose
Defines the canonical set of Swarm shard types used for parallel inference.
Shard types are standardized to: - prevent unsafe improvisation - ensure merge compatibility - keep parallelism deterministic
All shards are inference-only by default.

Core Rules
Each shard must have a single, narrow objective
Each shard must define a structured output schema
Shards may not execute tools unless explicitly allowed (default: no)
Shards may not access secrets
Shards may not approve methods or promote Talents
Related: - [[NOTION_PAGE:"[[Swarm Task Sharding|Swarm Task Sharding]]"]] - [[NOTION_PAGE:"[[Swarm Work Ticket|Swarm Work Ticket]]"]] - [[NOTION_PAGE:"[[Swarm Merge Rule|Swarm Merge Rule]]"]]

Canonical Shard Types
1) Requirements Extraction Shard
Purpose Extract explicit and implicit requirements from the prompt.
Input - user prompt - relevant chat slice
Output - list of requirements - constraints - non-goals - ambiguity list
Notes Often the first shard dispatched.

2) Intent Clarification Shard
Purpose Determine what the user actually wants vs what they literally asked.
Input - prompt - recent interaction context
Output - inferred intent - confidence score - clarification questions (if needed)
Notes Useful when prompt is vague or overloaded.

3) Method Draft Shard
Purpose Draft a candidate method that could satisfy the task.
Input - requirements artifact - constraints - task metadata
Output - step-by-step method outline - required tools (declared, not executed) - expected outputs - risk flags
Notes Does not authorize execution.

4) Risk Assessment Shard
Purpose Identify safety, privacy, or execution risks.
Input - method draft - task metadata
Output - risk list - severity per risk - mitigation suggestions - recommendation (safe / needs review)
Notes Feeds approval UX.

5) Tool Mapping Shard
Purpose Map method steps to required tools without executing them.
Input - method draft
Output - tool list - step → tool mapping - scope requirements
Notes Must not imply authorization.

6) Alternative Solution Shard
Purpose Generate alternative approaches or methods.
Input - requirements - constraints
Output - 1–3 alternative methods - pros / cons - tradeoffs
Notes Useful for complex or high-risk tasks.

7) Contradiction Detection Shard
Purpose Detect internal conflicts or contradictions.
Input - requirements - method draft - constraints
Output - contradiction list - explanation - severity
Notes Often paired with Method Draft.

8) Summarization Shard
Purpose Produce a concise summary of large context or artifacts.
Input - long chat slice - multiple shard outputs
Output - structured summary - key points - omissions
Notes Helps reduce merge complexity.

9) Acceptance Criteria Shard
Purpose Define what “done” means for the task.
Input - requirements - method draft
Output - acceptance criteria list - measurable outcomes
Notes Improves execution determinism.

10) UI Copy / UX Text Shard
Purpose Draft user-facing language (labels, prompts, explanations).
Input - task context - requirements
Output - UI strings - tone notes
Notes Never executable.

11) Architecture Consistency Check Shard
Purpose Ensure proposed method complies with existing atomic rules and MOCs.
Input - method draft - linked architecture notes
Output - compliance checklist - violations (if any) - suggested fixes
Notes Extremely useful in your system.

Optional / Advanced Shards
Performance Estimation Shard
estimates token usage
estimates latency
estimates energy impact
Merge Assistance Shard
suggests how shard outputs should be combined
flags overlap or redundancy

Shard Selection Guidance
Typical heavy prompt flow: 1) Requirements Extraction 2) Method Draft 3) Risk Assessment 4) Contradiction Detection 5) Acceptance Criteria
UI-heavy tasks often add: - UI Copy Shard
Architecture-heavy tasks add: - Architecture Consistency Check

Architectural Rule
Swarm parallelism increases intelligence, not authority. Execution authority remains singular.

## Related

^[source-materials/mirrors/doctrine/Swarm Shard Catalog.md]
