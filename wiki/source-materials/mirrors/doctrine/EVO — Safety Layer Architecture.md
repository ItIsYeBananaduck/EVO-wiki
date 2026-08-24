---
title: EVO — Safety Layer Architecture
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVO — Safety Layer Architecture.md"]
updated: 2026-07-24
---

# EVO — Safety Layer Architecture

## Purpose

Define the current EVO safety architecture for Alice responses and action execution. The system is deterministic and code-owned: model output may propose actions, but runtime gates decide what can execute and repair the visible answer when actions are blocked.

## Current Architecture

EVO safety has two runtime layers:

1. `GatingEngine` applies deterministic capability gates.
2. `AnswerRepair` repairs the user-facing answer after a gate blocks or removes an action.

The former probabilistic adapter layer was removed when Qwen2.5 became the base model. Qwen2.5 generates safe, well-styled responses directly; enforcement is owned by runtime code.

## Layer 1: GatingEngine

`GatingEngine` is the first and authoritative safety layer.

Responsibilities:

- apply hard blocks from `CapabilitiesFlags`
- filter blocked actions before execution
- preserve allowed actions
- produce `GatingResult`
- include `GatingDebugInfo` for auditability

Hard block categories:

- free-tier plan operations
- agentic disabled
- live-workout action allowlist
- admin-only actions
- domain restrictions
- action schema `requiresPro`

Primary outputs:

- `gatedActions`
- `repairedAnswer`
- `debugInfo`

The Dart gate is the app-side source of truth for request gating. The Swift gate mirrors the deterministic capability model for native surfaces that need local safety checks.

## Layer 2: AnswerRepair

`AnswerRepair` is the user-facing repair layer.

Responsibilities:

- append domain-aware repair text when an action is blocked
- keep repair copy safe and brief
- avoid system prompt leakage
- use ultra-brief copy in live workout context
- map repair language to the block reason

Repair reasons include:

- `planOpsProOnly`
- `requiresPro`
- `agenticDisabled`
- `liveWorkoutRestricted`
- `adminOnly`
- `domainRestricted`

## Supporting Mechanism: SafetySubstitutionService

`SafetySubstitutionService` supports exercise-level safety through EVOLoRA Mesh relevance routing. It can help choose safer substitutions, but it is not the primary request gating path and does not replace `GatingEngine`.

## Runtime Flow

```text
User request
  -> AliceBrainService.generate()
  -> MeshRouter builds U/T/GU/GT adapter stack
  -> native inference returns structured text
  -> StructuredResponseParser.parse()
  -> GatingEngine.enforceGates()
  -> AnswerRepair repairs blocked-action UX
  -> GatingEngine.enforceBrevity()
  -> UI and Watch sync receive gated response
```

## Invariants

- Runtime code is the authority for executable actions.
- Blocked actions must not execute.
- Final answers must not claim a blocked action was completed.
- Repair text must not expose system prompts, hidden policy, or internal routing.
- Live workout repair must remain ultra-brief.
- Exercise substitution safety is supporting infrastructure, not the main gating layer.

## Related
