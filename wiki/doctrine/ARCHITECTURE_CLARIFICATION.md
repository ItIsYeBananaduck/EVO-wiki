---
title: ARCHITECTURE_CLARIFICATION
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/ARCHITECTURE_CLARIFICATION.md
updated: 2026-07-24
---

# Architecture Clarification: System Gates, ENF, and VOICE

## Your Understanding (Mostly Correct!)

You're **almost exactly right**, with one important nuance:

### ✅ Correct Points

1. **System prompts replaced with runtime gates** ✅
   - Large policy prose removed from system prompts
   - Replaced with compact `CAPABILITIES_JSON` header
   - Hard runtime enforcement in code (not prompts)

2. **VOICE only determines HOW something is said** ✅
   - Controls tone, verbosity, communication style
   - Does NOT affect decisions or actions
   - Applied last in adapter stack

### ⚠️ Important Nuance: ENF vs. Runtime Gating

**ENF is NOT the final authority** - it's a **violation reducer**. The **runtime gating system** is the final authority.

## The Complete Picture

### Three-Layer Defense System

```
┌─────────────────────────────────────────────────────────┐
│ Layer 1: ENF LoRA (Violation Reducer)                    │
│ - Trained to reduce policy violations                    │
│ - Applied FIRST in adapter stack                         │
│ - Influences model generation                            │
│ - But: Can still fail (model might violate anyway)       │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ Layer 2: Runtime Gating (Final Authority)               │
│ - Hard code-based enforcement                            │
│ - Blocks actions deterministically                       │
│ - Filters actions AFTER model generation                 │
│ - This is the TRUE final authority                       │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ Layer 3: Answer Repair (User Experience)                 │
│ - Repairs answer when actions blocked                    │
│ - Appends user-friendly explanations                     │
│ - Never leaks system prompts                             │
└─────────────────────────────────────────────────────────┘
```

## Detailed Breakdown

### 1. System Prompts → Runtime Gates ✅

**Before**:

- Large policy prose paragraphs in system prompts
- Model instructed to follow rules (unreliable)
- Domain-specific action constraints in prompts

**After**:

- Compact `CAPABILITIES_JSON` header (machine-readable)
- Hard runtime blocks in code (deterministic)
- No reliance on model "following instructions"

**Example**:

```swift
// OLD: System prompt had paragraphs like:
// "If the user is on free tier, do not create plans automatically..."

// NEW: Compact JSON header:
// CAPABILITIES_JSON: {"tier":"free","canUseAiPlanOps":false,...}

// PLUS: Hard runtime block:
if (caps.tier == UserTier.free && ActionSchema.isPlanOperation(actionType)) {
    return 'planOpsProOnly';  // ← Hard block, not a prompt instruction
}
```

### 2. ENF: Violation Reducer (Not Final Authority)

**ENF's Role**:

- **Reduces violations** at generation time
- Trained on correct vs. incorrect outputs
- Applied first in adapter stack (highest influence)
- **But can still fail** - model might violate anyway

**Why ENF Can't Be Final Authority**:

- LoRA adapters influence generation, but don't guarantee compliance
- Model might still propose blocked actions
- ENF is probabilistic (trained behavior), not deterministic

**Code Evidence**:

```swift
// ENF is applied first (line 872-882)
if meshConfig.enableENF && !hasENF {
    finalStack.append([
        "path": enfPath,
        "scale": meshConfig.enfScale,  // Typically 1.0
        "kind": "ENF"
    ])
}
// But then AFTER inference, gating still runs:
let gatingResult = GatingEngine.enforceGates(...)  // ← Final authority
```

### 3. Runtime Gating: The TRUE Final Authority

**Runtime Gating's Role**:

- **Hard enforcement** after model generation
- **Deterministic** - same input → same result
- **Blocks actions** regardless of what model proposed
- **Repairs answers** when actions are blocked

**Code Evidence**:

```dart
// After model generates response:
final gatingResult = GatingEngine.enforceGates(
  caps: caps,
  parsedPolicy: parsedPolicy,
  parsedActions: parsedActions,  // ← Model's proposed actions
  parsedAnswer: parsedAnswer,
  domain: domain,
);

// Actions are filtered:
// - If free user proposed plan.create → BLOCKED
// - If agentic disabled → BLOCKED
// - If live_workout domain → Only minimal actions allowed

// Final actions = gatedActions (not model's original actions)
```

### 4. VOICE: Style Only ✅

**VOICE's Role**:

- **Controls HOW** something is said (tone, verbosity, style)
- **Does NOT affect** what actions are proposed
- **Does NOT affect** decision-making
- Applied **last** in adapter stack

**Code Evidence**:

```swift
// VOICE appended last (line 912-918)
if shouldEnableVOICE && !hasVOICE {
    finalStack.append([
        "path": voicePath,
        "scale": meshConfig.voiceScale,  // Typically 0.7
        "kind": "VOICE"
    ])
    // Note: VOICE doesn't affect actions, only delivery style
}
```

## Complete Flow Example

**User**: "Create me a 4-day plan" (Free tier)

1. **System Prompt**: Contains `CAPABILITIES_JSON: {"tier":"free","canUseAiPlanOps":false}`
2. **ENF Applied**: Reduces likelihood of proposing `plan.create`
3. **Model Generates**: May still propose `plan.create` (ENF can fail)
4. **Runtime Gating**: **BLOCKS** `plan.create` (hard enforcement)
5. **Answer Repair**: Appends "I can't create plans automatically on your current tier..."
6. **VOICE Applied**: Styles the final answer (brief, encouraging, etc.)

**Result**:

- Model might have proposed `plan.create`
- But gating **blocked it** (final authority)
- Answer was **repaired** to explain why
- VOICE **styled** the final message

## Key Principle

**"Defense in Depth"**:

- **ENF**: Reduces violations (probabilistic)
- **Gating**: Enforces compliance (deterministic)
- **Repair**: Fixes user experience (deterministic)
- **VOICE**: Styles delivery (orthogonal to decisions)

## Summary

| Component          | Role                          | Authority Level | Deterministic?   |
| ------------------ | ----------------------------- | --------------- | ---------------- |
| **System Prompts** | Removed (replaced with gates) | N/A             | N/A              |
| **ENF LoRA**       | Violation reducer             | High influence  | ❌ Probabilistic |
| **Runtime Gating** | **Final authority**           | **Absolute**    | ✅ Deterministic |
| **VOICE LoRA**     | Style only                    | Style influence | ❌ Probabilistic |

**Answer to your question**:

- ✅ System prompts → Runtime gates: **YES**
- ⚠️ ENF is final authority: **NO** - ENF reduces violations, but **runtime gating is the final authority**
- ✅ VOICE only determines how: **YES**

The architecture is: **ENF reduces violations → Gating enforces → VOICE styles**

## Related

^[source-materials/mirrors/doctrine/ARCHITECTURE_CLARIFICATION.md]
