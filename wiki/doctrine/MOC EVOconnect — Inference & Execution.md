---
title: MOC EVOconnect — Inference & Execution
type: concept
tags: [connect, evo, execution, inference, moc]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# MOC EVOconnect — Inference & Execution

Inference & Execution MOC
Purpose
This MOC maps how Alice performs inference, manages execution contexts, and delivers insights while respecting privacy, energy constraints, and user attention.
This note contains no rules. It only links to them.

Core Principles
Foundational constraints that govern all execution behavior.
[On-Device First Principle](https://www.notion.so/On-Device-First-Principle-33ec72bad0138100bbe3ebc5c290f7b8)
[Energy-Aware Inference](https://app.notion.com/p/33ec72bad013818aa564f83af05fb2ba)
[Warm State Preservation](https://www.notion.so/Warm-State-Preservation-33ec72bad01381638043ee679eba3da1)

Execution Contexts
Alice adapts behavior based on system context.
Foreground Execution
User-visible interaction
Direct responses
Explanations when requested
[Presence by Value](https://app.notion.com/p/33ec72bad0138180b6f7c3340e5ef148)
[Explain Only When Asked](https://app.notion.com/p/33ec72bad0138119bfa0e007260ebea7)

Background Execution
Silent processing only
State updates and aggregation
No user-facing output
[Background Inference Rules](https://app.notion.com/p/33ec72bad01381e4861ae0e04fc67d34)
[Deferred Response Strategy](https://www.notion.so/Deferred-Response-Strategy-33ec72bad0138165a14beefbab3fecc1)

Live Activity Execution
Pocket-safe operation
Minimal UI updates
Interruptions only for safety
[Live Activity Inference](https://app.notion.com/p/33ec72bad01381c78f2ff2ab2e8dab34)
[Non-Intrusive Guidance](https://www.notion.so/Non-Intrusive-Guidance-33ec72bad013810e9792e13c03de6618)

Timing & Delivery
Controls when insights are surfaced.
[Deferred Response Strategy](https://www.notion.so/Deferred-Response-Strategy-33ec72bad0138165a14beefbab3fecc1)
[Silence as a Feature](https://app.notion.com/p/33ec72bad01381e9ad13e6d6c96ea670)
[Explain Only When Asked](https://app.notion.com/p/33ec72bad0138119bfa0e007260ebea7)

Relationship to Alice’s Presence
Execution rules shape visibility and tone.
[Presence by Value](https://app.notion.com/p/33ec72bad0138180b6f7c3340e5ef148)
[Alice Visibility Rules](https://app.notion.com/p/33ec72bad01381cda0d9f5eb01c608bb)
[Tone Follows Context](https://app.notion.com/p/33ec72bad0138133871bdc52c517ce66)

Relationship to Training Intelligence
Execution constraints bound adaptation logic.
[Risk-Weighted Adaptation](https://www.notion.so/Risk-Weighted-Adaptation-33ec72bad013819eabb4c59900b06708)
[Mesocycle Awareness](https://app.notion.com/p/33ec72bad01381a9ac19f53a641a29d5)
[Adaptive Variable Adjustment](https://www.notion.so/Adaptive-Variable-Adjustment-33ec72bad01381e08c75cba76b076b7f)

Safety & Trust Boundaries
Execution behavior enforces trust implicitly.
[Cold Start Safety](https://www.notion.so/Cold-Start-Safety-33ec72bad01381f89f2fd681aaefedcd)
[User LoRA Silence Condition](https://www.notion.so/User-LoRA-Silence-Condition-33ec72bad01381659ff6c71ceb17ef6c)
[Authority vs Influence](https://www.notion.so/Authority-vs-Influence-33ec72bad01381979a74f7600ad56c70)

Usage Notes
Start here when reasoning about performance or energy behavior
Add new execution contexts only when a new runtime mode exists
Do not place rules in this note

Architectural Rule
If a rule exists, it belongs in an atomic note. If it connects rules, it belongs in a MOC.

## Related
- [[EVOconnect — System Map]]
- [[EVO Architecture Bible]]
- [[MOC EVOconnect — Agent System.md]]
- [[MOC EVOconnect — Cognitive & Execution Model.md]]
- [[MOC EVOconnect — Environment Integrations.md]]
- [[MOC EVOconnect — Escalation & Delegation.md]]
- [[MOC EVOconnect — Methods & Talents.md]]
- [[MOC EVOconnect — Task Lifecycle.md]]
^[wiki-native — no upstream source]
