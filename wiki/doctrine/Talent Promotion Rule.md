---
title: Talent Promotion Rule
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Talent Promotion Rule.md"]
updated: 2026-07-24
---

# Talent Promotion Rule
### Concept

Only proven-safe methods may become Talents.

### Rule / Mechanism

A method may be promoted to a Talent only if:

-  It has been executed successfully with user approval - No safety violations occurred
- The user explicitly consents to reuse without per-task approval

Promotion freezes the method definition.

### Why It Exists

Trust must be earned and preserved.
Implications
Talents are stable
Talents cannot evolve silently
Future changes require a new method and re-promotion

## Promotion Consent

Talent promotion requires explicit user consent.

Even when a Method meets all validation requirements:

- successful execution
- no safety violations
- repeated success

It must not be promoted unless the user explicitly agrees to preserve it.

This ensures:

- user control over preserved behavior
- no unwanted automation
- alignment with user intent

Task Chain verification may enable automatic promotion pathways, but user-facing automation must still be surfaced transparently.

## Promotion Threshold

A Method becomes eligible for promotion after:

- **3 successful validated executions**

A successful execution requires:

- user confirmation
- no safety violations
- correct outcome

Once this threshold is met:

- Alice prompts the user for promotion
- promotion is not automatic unless governed by a verified Task Chain

This threshold defines the minimum proof required for a Method to be considered stable.

### Links

[Method Approval Path](https://www.notion.so/33ec72bad01381fa9b3ec4729d474082)
[Talent Definition](https://www.notion.so/33ec72bad0138124922ee770d3aebbc0)
User Override Supremacy

## Related

^[{src_rel}]
