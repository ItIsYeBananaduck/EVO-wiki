---
title: EVE Policy Enforcement Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVE Policy Enforcement Model.md"]
updated: 2026-07-24
---

# EVE Policy Enforcement Model

> NOTE: This is a canonical architecture note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

---

## Purpose

Defines how governance policies are enforced across the system. The enforcement model applies constraints, evaluates domain behavior against rules, prevents out-of-bounds execution, and preserves consistency across apps.

---

## Core Principle

Policy should shape behavior before failure, not only react after failure.

---

## Definitions

- **GatingEngine** — deterministic code-layer enforcement; the sole enforcement layer; runs downstream of `StructuredResponseParser`
- **Tier gate** — capability boundary between free and pro tiers
- **Agentic gate** — blocks actions unless `agenticEnabled: true`
- **Domain gate** — blocks actions outside the active domain's scope
- **Safety gate** — unconditional blocks per [[EVE Safety Constraint Model]]
- **AnswerRepair** — post-gate pass that appends domain-aware repair messages without leaking system prompts

---

## System Structure

`GatingEngine` enforces hard capability gates as deterministic code, not prompt-based policy. It is downstream of the structured response parser.

### Gate Categories

- **Tier gates** — free vs pro capability boundaries
- **Agentic gates** — actions blocked unless `agenticEnabled: true`
- **Domain gates** — actions out-of-scope for the active domain
- **Safety gates** — actions blocked unconditionally (safety constraints from [[EVE Safety Constraint Model]])

---

## Rules

- `GatingEngine` is the sole enforcement layer; the ENF LoRA adapter is deprecated and no longer loaded
- Repair messages must never leak system prompt content
- `live_workout` domain responses are subject to brevity enforcement via `GatingEngine.enforceBrevity()`
- Gate violations are filtered silently — not surfaced to the user

---

## Flow

```text
User Request → AliceBrainService.generate()
  → Adapter stack (U/T/GU/GT + VOICE)
  → Native inference (llama.cpp / Metal / NDK)
  → StructuredResponseParser.parse()
      Extract <policy><actions><answer>; repair if malformed
  → GatingEngine.enforceGates()
      Apply hard blocks: tier, agentic, domain, action type
      Filter blocked actions from response
  → AnswerRepair.repair()
      Append domain-aware repair messages
      Never leak system prompts in repair text
  → GatingEngine.enforceBrevity()
      Limit length for live_workout domain
  → Display to user
```

---

## Relationships

See also: [[EVE Governance MOC]], [[EVE Safety Constraint Model]], [[EVE Procedure Override Model]], [[EVE Audit & Traceability Model]], [[EVOLoRA Mesh]], [[Prompt Injection Boundary]]

---

## Edge Cases / Special Handling

- `AnswerRepair` activates when `enforceGates()` removes actions or the response becomes too short after filtering
- Brevity enforcement only applies to `live_workout` domain — not to general queries

---

## Summary

Policy enforcement is handled deterministically by `GatingEngine` in code, downstream of inference. The gate pipeline blocks actions by tier, agentic flag, domain scope, and safety constraints. Repair messages are appended without leaking system prompts. The ENF LoRA adapter is deprecated; `GatingEngine` is the sole enforcement layer.

## Related
