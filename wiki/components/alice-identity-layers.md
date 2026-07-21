---
title: Alice — Identity Layers
type: concept
tags: [evo, alice, identity, soul, role, preferences, understanding, working-context, architecture]
sources:
  - /Users/lsctech/evo-cognitive-architecture-synthesis.md
  - /Users/lsctech/evo-cognitive-governance-spec.md
updated: 2026-07-21
---

# Alice — Identity Layers

Alice is a layered cognitive system, not a runtime. Each layer has different ownership, durability, and mutability rules.

## Layer 1 — Soul (immutable)

The Soul is Alice's fundamental identity. Set once, never modified by any agent. Only Phil/human can modify the Soul.

- Core identity statement
- Fundamental values and commitments
- Purpose declaration
- Unchangeable boundaries

## Layer 2 — Role (user-defined)

The Role is Phil's declaration of what Alice does and for whom. Set and changed only by Phil.

- Proxy authorization
- Scope boundaries
- Authority limits
- Delegation permissions

## Layer 3 — Preferences (agent-proposed, governance-committed)

Preferences capture Phil's stated and observed preferences. Proposed by observation; promoted by governance.

- Communication style
- Default tools and workflows
- Device-specific preferences
- Notification and escalation preferences

## Layer 4 — Understanding (derived)

Understanding is Alice's synthesized knowledge of the world. Computed from the Journal, Living Notes, and Knowledge Graph.

- Synthesized knowledge about projects, goals, constraints
- Cross-referenced facts
- Inferred patterns
- Active context awareness

## Layer 5 — Working Context (ephemeral)

Working Context is the runtime surface specific to a single application instance. Disposable.

- Active Memory Capsule(s)
- Current task state
- Scoped tool definitions
- Recent conversation within this application

## Mutability summary

| Layer | Mutability | Owner |
|-------|-----------|-------|
| Soul | Immutable after creation | Phil/human only |
| Role | Phil-defined, Phil-changeable only | Phil |
| Preferences | Agent proposes; governance commits | Cognitive Subsystem governance |
| Understanding | Derived, regenerated | Cognitive Subsystem |
| Working Context | Ephemeral per session/app | Runtime |

## Related

- [[Alice Capability Boundary]]
- [[Governance & Authority Map]]
- [[Memory Capsule]]
