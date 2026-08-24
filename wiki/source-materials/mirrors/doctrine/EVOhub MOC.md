---
title: EVOhub MOC
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOhub MOC.md"]
updated: 2026-07-24
---

# EVOhub MOC
EVOhub MOC
Purpose
EVOhub is the coordination and integration layer across all EVO applications.
It does not introduce intelligence. It routes, connects, and contextualizes it.

Core Responsibilities
EVOhub is responsible for:
Cross-app awareness
Shared identity (Alice)
Context handoff between domains
Unified user experience

What EVOhub Is NOT
Not a new AI model
Not a rules engine
Not a decision maker
All intelligence remains in domain systems.

Shared Identity
EVOhub ensures Alice feels consistent across:
EVOtraining
EVOmind
EVOlearn
Future EVO apps
Related: - [[Alice (AI Companion)]] - [Tone Follows Context](https://app.notion.com/p/Tone-Follows-Context-33ec72bad0138133871bdc52c517ce66)

Context Routing
EVOhub manages transitions such as:
Training → Recovery → Reflection
Learning → Practice → Feedback
Mental state influencing training tone
Routing respects: - [Presence by Value](https://app.notion.com/p/Presence-by-Value-33ec72bad0138180b6f7c3340e5ef148) - [Non-Intrusive Guidance](https://www.notion.so/33ec72bad013810e9792e13c03de6618)

Data Boundaries
EVOhub does not centralize raw data.
On-device context only
Minimal shared summaries
No unnecessary cross-domain leakage
Related: - [On-Device First Principle](https://www.notion.so/33ec72bad0138100bbe3ebc5c290f7b8)

Relationship to Execution
EVOhub operates above execution layers.
[[Inference & Execution MOC]]

Open Design Questions
Unresolved architectural decisions.
How explicit cross-app transitions should be
How much context should flow between domains
Whether EVOhub has a visible UI or remains implicit

Architectural Rule
EVOhub coordinates intelligence. It never competes with it.

## Related
