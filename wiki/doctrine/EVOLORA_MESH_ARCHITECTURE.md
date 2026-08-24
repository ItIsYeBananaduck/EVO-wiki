---
title: EVOLORA_MESH_ARCHITECTURE
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVOLORA_MESH_ARCHITECTURE.md
updated: 2026-07-24
---

# EVOLoRA Mesh Architecture

**Adaptive LoRA Adapter System for Alice AI Coach**

---

## Overview

EVOLoRA Mesh is a **dynamic LoRA (Low-Rank Adaptation) adapter system** that enables Alice to adapt her behavior based on context, user preferences, and trainer guidance. Instead of retraining the entire model, we use lightweight LoRA adapters that modify the base model's behavior at inference time.

**Key Principle**: The base model (`alice-human-fusion.Q4_K_M.gguf`) remains static, while multiple LoRA adapters are blended together to create contextually appropriate responses.

---

## Architecture Components

### 1. Base Model

- **Model**: `alice-human-fusion.Q4_K_M.gguf` (7B parameters, 4-bit quantized)
- **Size**: ~2.4GB
- **Runtime**: llama.cpp with Metal (iOS) / NDK (Android)
- **Purpose**: Foundation model that provides general fitness coaching knowledge

### 2. LoRA Adapter Types

The system supports six types of LoRA adapters:

#### **U (User Adapter)**

- **Purpose**: Personalizes responses based on individual user preferences and history
- **Training**: On-device, nightly QLoRA training using user's workout data
- **Scope**: User-specific behavior patterns
- **Size**: ~50MB
- **Location**: `AliceAssets/adapters/user/user_lora.gguf`

#### **T (Trainer Adapter)**

- **Purpose**: Applies trainer-specific coaching style when user has an assigned trainer
- **Training**: Trainer's coaching patterns and preferences
- **Scope**: Per-trainer (one adapter per trainer)
- **Size**: ~50MB
- **Location**: `AliceAssets/adapters/trainer/{trainerId}_lora.gguf`

#### **GU (Global User Adapter)**

- **Purpose**: Aggregated learnings from all users (federated learning)
- **Training**: Weekly aggregation of user deltas on Fly.io backend
- **Scope**: Global user patterns
- **Size**: ~100MB
- **Location**: `AliceAssets/adapters/global/global_user_lora.gguf`

#### **GT (Global Trainer Adapter)**

- **Purpose**: Aggregated learnings from all trainers
- **Training**: Weekly aggregation of trainer deltas
- **Scope**: Global trainer patterns
- **Size**: ~100MB
- **Location**: `AliceAssets/adapters/global/global_trainer_lora.gguf`

#### **ENF (Enforcer Adapter)**

- **Purpose**: **Policy enforcement layer** - ensures Alice follows output contracts and safety guidelines
- **Training**: Trained on violation examples and correct output formats
- **Scope**: Universal (applied to all requests)
- **Size**: ~25MB
- **Location**: `AliceAssets/adapters/enforcer/enforcer_lora.gguf`
- **Priority**: **Always applied first** in adapter stack (highest authority)

#### **VOICE (Voice Style Adapter)**

- **Purpose**: **Delivery style only** - controls tone, verbosity, and communication style (NOT decision-making)
- **Training**: Trained on different communication styles (brief, encouraging, technical, etc.)
- **Scope**: Context-dependent (disabled for `live_workout` domain)
- **Size**: ~15MB
- **Location**: `AliceAssets/adapters/voice/voice_lora.gguf`
- **Note**: VOICE does NOT affect what actions Alice proposes, only HOW she communicates

---

## How EVOLoRA Mesh Works

### Decision Flow

```
User Request
    ↓
MeshRouter.determineContext() → Maps domain/action to MeshContext
    ↓
MeshEngine.select() → Computes LoRAContribution for each adapter type
    ↓
LoRAAdapterManager → Resolves adapter file paths
    ↓
Adapter Stack Built → List of adapters with weights (scales)
    ↓
Native Layer (iOS/Android) → Applies adapters to base model
    ↓
Inference → Model generates response with blended behavior
```

### MeshContext Types

The system maps user requests to contexts that determine which adapters are relevant:

- **`executionMicroadjust`**: Active workout - real-time adjustments
- **`weeklyOverloadDecision`**: Weekly planning - progression decisions
- **`planCreateMajor`**: Creating new workout plans
- **`nutritionAdjust`**: Nutrition guidance and adjustments
- **`recoveryGuidance`**: Recovery and rest recommendations
- **`generalQuery`**: General fitness questions

### Adapter Stack Construction

The `MeshRouter` builds an adapter stack by:

1. **Determining Context**: Maps domain/action to `MeshContext`
2. **Getting Contributions**: Calls `MeshEngine.select()` to get weighted contributions
3. **Resolving Paths**: Uses `LoRAAdapterManager` to find local adapter files
4. **Building Stack**: Creates list of adapters with scales (0.0 - 2.0)
5. **Adding ENF**: Always inserts ENF adapter at position 0 (highest priority)

**Example Adapter Stack**:

```dart
[
  {kind: "ENF", path: ".../enforcer_lora.gguf", scale: 1.0},  // Always first
  {kind: "U", path: ".../user_lora.gguf", scale: 0.7},        // User personalization
  {kind: "GU", path: ".../global_user_lora.gguf", scale: 0.3}, // Global patterns
  {kind: "VOICE", path: ".../voice_lora.gguf", scale: 0.7},    // Style only
]
```

---

## ENF (Enforcer) LoRA

### Purpose

ENF is a **policy enforcement layer** that ensures Alice:

- Follows structured output format (`<policy><actions><answer>`)
- Doesn't claim to execute blocked actions
- Respects capability constraints
- Maintains safety guidelines

### How It Works

1. **Always Applied First**: ENF adapter is inserted at position 0 in the adapter stack
2. **High Authority**: Scale of 1.0 (full influence)
3. **Reduces Violations**: Trained on examples of correct vs. incorrect outputs
4. **Works with Gating**: ENF reduces violations, but **hard gating** (runtime enforcement) is the final safety net

### Training Data

ENF is trained on:

- Examples of correct structured output
- Violation cases (model claiming blocked actions, breaking format)
- Repair examples (how to fix violations)

### Integration with Gating System

```
┌─────────────────────────────────────────┐
│  Model Inference (with ENF adapter)    │
│  ↓                                      │
│  Generates response                     │
│  ↓                                      │
│  parseStructuredResponse()              │
│  ↓                                      │
│  applyHardGatingToActions()            │ ← Runtime enforcement
│  ↓                                      │
│  repairAnswerForBlockedActions()       │ ← Deterministic repair
└─────────────────────────────────────────┘
```

**Key Point**: ENF reduces violations, but **gating is the hard enforcement**. Even if ENF fails, runtime gating blocks illegal actions.

---

## VOICE LoRA

### Purpose

VOICE controls **delivery style only** - tone, verbosity, and communication approach. It does **NOT** affect:

- What actions Alice proposes
- Decision-making logic
- Policy enforcement

### How It Works

1. **Style Modulation**: Adjusts how Alice communicates (brief, encouraging, technical, etc.)
2. **Context-Aware**: Disabled for `live_workout` domain (ultra-brief mode)
3. **Lower Priority**: Applied after ENF and decision adapters
4. **Scale**: Typically 0.7 (moderate influence)

### Use Cases

- **Live Workout**: VOICE disabled → Ultra-brief responses
- **Planning**: VOICE enabled → More detailed, encouraging tone
- **Recovery**: VOICE enabled → Supportive, gentle tone
- **Nutrition**: VOICE enabled → Educational, clear tone

### Training Data

VOICE is trained on:

- Different communication styles (brief, detailed, technical, friendly)
- Tone variations (encouraging, neutral, supportive)
- Verbosity levels (short, medium, long)

---

## Integration with Capability & Gating System

The gating system works **orthogonally** to EVOLoRA Mesh:

### Separation of Concerns

1. **EVOLoRA Mesh (LoRAs)**: Controls **behavior and style**
   - ENF: Policy compliance
   - VOICE: Communication style
   - U/T/GU/GT: Personalization and decision-making

2. **Gating System**: Controls **action execution**
   - Hard runtime blocks
   - Capability-based restrictions
   - Answer repair

### How They Work Together

```
┌─────────────────────────────────────────────────────────┐
│  1. User Request                                         │
│     ↓                                                    │
│  2. MeshRouter builds adapter stack (ENF + U + VOICE)  │
│     ↓                                                    │
│  3. Native inference with adapters                      │
│     ↓                                                    │
│  4. Parse structured output                             │
│     ↓                                                    │
│  5. Gating Engine enforces capabilities                 │ ← Hard enforcement
│     ↓                                                    │
│  6. Repair answer if actions blocked                    │
│     ↓                                                    │
│  7. Return gated response                               │
└─────────────────────────────────────────────────────────┘
```

**Key Principle**:

- **LoRAs influence what the model generates** (behavior/style)
- **Gating enforces what gets executed** (safety/capabilities)

---

## Adapter Application Order

The native layer (iOS/Android) applies adapters in this order:

1. **ENF** (scale: 1.0) - Policy enforcement
2. **U** (scale: 0.7) - User personalization
3. **T** (scale: 0.5) - Trainer style (if applicable)
4. **GU** (scale: 0.3) - Global user patterns
5. **GT** (scale: 0.2) - Global trainer patterns
6. **VOICE** (scale: 0.7) - Communication style

**Note**: Scales are computed by `MeshEngine` based on context and user state.

---

## File Structure

```
AliceAssets/
├── models/
│   └── alice-human-fusion.Q4_K_M.gguf  (Base model)
└── adapters/
    ├── enforcer/
    │   └── enforcer_lora.gguf
    ├── voice/
    │   └── voice_lora.gguf
    ├── user/
    │   └── user_lora.gguf
    ├── trainer/
    │   └── {trainerId}_lora.gguf
    └── global/
        ├── global_user_lora.gguf
        └── global_trainer_lora.gguf
```

---

## Current Implementation Status

### ✅ Completed

- **MeshRouter**: Routes requests to appropriate adapters
- **LoRAAdapterManager**: Manages adapter file paths and metadata
- **ENF Integration**: ENF adapter always included in stack
- **VOICE Integration**: VOICE adapter included when appropriate
- **Gating System**: Hard enforcement after adapter inference
- **iOS Support**: Full adapter stack support in LlamaEngine.swift
- **Android Support**: Adapter stack support in LlamaPlugin.kt

### 🔄 In Progress

- **On-Device Training**: Nightly QLoRA training for U adapters
- **Federated Aggregation**: Weekly GU/GT adapter updates
- **ENF Training**: Training ENF on violation examples
- **VOICE Training**: Training VOICE on style variations

### 📋 Planned

- **Adapter Versioning**: Support for multiple adapter versions
- **Dynamic Download**: Auto-download missing adapters
- **Adapter Validation**: Checksum verification and integrity checks
- **Performance Optimization**: Caching and preloading strategies

---

## Key Design Decisions

### Why LoRA Instead of Full Fine-Tuning?

1. **Efficiency**: LoRA adapters are 10-100MB vs. 2.4GB base model
2. **Modularity**: Mix and match adapters for different contexts
3. **Privacy**: User adapters stay on-device
4. **Speed**: Faster inference with smaller adapter files

### Why ENF as a LoRA?

1. **Consistency**: Same mechanism as other adapters
2. **Flexibility**: Can update ENF without retraining base model
3. **Effectiveness**: Reduces violations at generation time
4. **Safety**: Works with hard gating for defense in depth

### Why VOICE Separate from Decision LoRAs?

1. **Separation of Concerns**: Style vs. content
2. **Independent Updates**: Can update VOICE without affecting decisions
3. **Context Control**: Can disable VOICE for ultra-brief domains
4. **Clarity**: Makes it explicit that VOICE doesn't affect actions

---

## Example: Request Flow

**User**: "Create me a 4-day hypertrophy plan" (Free tier user)

1. **MeshRouter** determines context: `planCreateMajor`
2. **MeshEngine** computes contributions:
   - ENF: 1.0 (always)
   - U: 0.7 (user preferences)
   - GU: 0.3 (global patterns)
   - VOICE: 0.7 (planning style)
3. **Adapter Stack** built and sent to native layer
4. **Native Inference** generates response with adapters
5. **Parsing** extracts `<policy><actions><answer>`
6. **Gating Engine** blocks `plan.create` action (Free tier)
7. **Answer Repair** appends: "I can't do plan creation automatically on your current tier..."
8. **Response** returned with gated actions and repaired answer

**Result**: Model may have proposed `plan.create`, but gating blocked it and answer was repaired.

---

## Summary

EVOLoRA Mesh is a **modular, adaptive system** that:

- Uses lightweight LoRA adapters to modify behavior
- Separates concerns (ENF for policy, VOICE for style, U/T for decisions)
- Works with hard gating for safety
- Enables personalization without full model retraining
- Maintains privacy (user adapters stay on-device)

**ENF** ensures policy compliance, **VOICE** controls communication style, and **gating** enforces capabilities - all working together to create a safe, personalized, and contextually appropriate AI coach.

## Related

^[source-materials/mirrors/doctrine/EVOLORA_MESH_ARCHITECTURE.md]
