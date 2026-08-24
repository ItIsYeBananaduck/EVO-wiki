---
title: EVOLoRA Mesh
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/EVOLoRA Mesh.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh

---

## Purpose

Define the dynamic LoRA adapter blending system that shapes Alice's inference behavior. The Mesh selects and weights adapters based on authority and relevance, enabling safe personalization, gradual cold-start behavior, and conflict resolution without static rules.

---

## Core Principle

`effective_weight = authority_weight × relevance`

LoRAs with zero relevance have zero influence regardless of authority. Adapters learn how to interpret memory, not store it. The EVO Wiki remembers; the Mesh interprets; Alice decides.

---

## Definitions

- **effective_weight** — `authority_weight × relevance`; the computed contribution of each adapter to inference
- **MeshRouter** — builds the adapter stack per request based on domain and action context
- **MeshContext** — describes the request type (e.g., `executionMicroadjust`, `weeklyOverloadDecision`, `planCreateMajor`, `nutritionAdjust`, `recoveryGuidance`, `generalQuery`)
- **MeshEngine** — computes weighted contributions per adapter type
- **LoRAAdapterManager** — resolves local GGUF file paths for loaded adapters

---

## System Structure

### Adapter Types

Four adapter types are loaded at runtime from llama.cpp-compatible GGUF files:

| Adapter | Size | Authority | Training | Location |
|---|---|---|---|---|
| **GU** (Global User) | ~100 MB | High | Weekly federated aggregation (Fly.io) | `AliceAssets/adapters/global/global_user_lora.gguf` |
| **GT** (Global Trainer) | ~100 MB | High | Weekly trainer-delta aggregation | `AliceAssets/adapters/global/global_trainer_lora.gguf` |
| **U** (User) | ~50 MB | Medium | On-device nightly QLoRA | `AliceAssets/adapters/user/user_lora.gguf` |
| **T** (Trainer) | ~50 MB | Medium | Trainer coaching patterns | `AliceAssets/adapters/trainer/{trainerId}_lora.gguf` |

> The former enforcement and style adapters were removed when the project migrated to Qwen2.5. The base model generates safe, well-styled responses directly. Policy enforcement is handled by `GatingEngine` and answer repair in code; see [[EVE Policy Enforcement Model]].

### Base Model

`alice-human-fusion.Q4_K_M.gguf` — ~2.4 GB, 7B parameters, 4-bit quantized. Runtime: llama.cpp with Metal (iOS) / NDK (Android). The base model remains static; adapters modify behavior at inference time only.

### Adapter Scope Model

Each adapter category is small and scoped:

- **Domain Adapter** — training / mind / learn / connect behavior
- **Global User Adapter** — population-level response trends
- **User Adapter** — individual response tendencies
- **Global Trainer Adapter** — cross-trainer outcome patterns
- **Trainer Adapter** — trainer-specific programming tendencies

### Memory Responsibility Split

| Layer | Role |
|---|---|
| Logs | Raw evidence |
| Summaries | Compressed evidence |
| Wiki | Structured memory |
| Adapters | Interpretation bias |

### Stack Construction

`MeshRouter` builds the stack per request:

1. Map domain/action → `MeshContext`
2. `MeshEngine.select()` computes weighted contributions per adapter type
3. `LoRAAdapterManager` resolves local GGUF file paths
4. Stack built as `[(adapter_path, scale: 0.0–2.0), ...]`
5. Stack passed to llama.cpp runtime

---

## Rules

- Adapters must remain as small as possible — only learn stable interpretation patterns that are repeated, useful, scope-appropriate, and difficult to express through prompt/context alone
- Adapters must NOT store: raw logs, chat history, full plans, journal entries, wiki pages, sensitive cross-app source content
- Adapter relevance may be influenced by: active app/domain, current task/session, physical/emotional/cognitive stress signals, wiki summaries, cross-app synthesis patterns, device capacity

---

## Flow

At runtime, Alice loads relevant adapters (based on domain + context), wiki summaries, and current signals. Adapters influence interpretation. Wiki provides factual grounding. Alice produces the decision.

Personalization is split into two layers:
- **Explicit** — EVO Wiki, summaries, logs, scoped memory pages
- **Implicit** — lightweight adapter interpretation bias

Different EVO apps reuse the same Alice intelligence model while loading different domain adapters and wiki summaries. Alice remains one intelligence moving through many app rooms.

---

## Relationships

See also: [[Authority vs Influence]], [[User LoRA Silence Condition]], [[Cold Start Safety]], [[EVOLoRA Mesh — Adapter Creation Pipeline]], [[LoRA Artifact Sync]], [[Model Storage Architecture — R2]], [[EVE Policy Enforcement Model]]

---

## Edge Cases / Special Handling

- Cold start: [[User LoRA Silence Condition]] governs when U/T adapters are suppressed during early sessions
- Device constraints: >4 adapters requires active RAM management
- Adapter update: pull updated GGUF from R2; re-initialization happens on next session start

---

## Summary

The EVOLoRA Mesh blends up to four LoRA adapter types (GU, GT, U, T) using `effective_weight = authority_weight × relevance`. Adapters learn interpretation bias only — structured memory lives in the EVO Wiki. `MeshRouter` builds the stack per request via `MeshContext`. Policy enforcement is owned by `GatingEngine` and answer repair. One Alice intelligence model operates across all EVO apps by loading different domain adapters and wiki summaries.

## Related
