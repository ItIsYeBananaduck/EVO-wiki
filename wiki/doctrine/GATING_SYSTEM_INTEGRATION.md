---
title: GATING_SYSTEM_INTEGRATION
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/GATING_SYSTEM_INTEGRATION.md
updated: 2026-07-24
---

# Gating System Integration Guide

**Checks & Balances Runtime Gating for EVOLoRA Mesh**

---

## Overview

This document describes how the runtime gating system integrates with the existing EVOLoRA Mesh architecture. The gating system provides **hard runtime enforcement** of capabilities, replacing policy-heavy system prompts with deterministic code-based rules.

---

## Architecture Flow

```
┌─────────────────────────────────────────────────────────────┐
│  User Request (Flutter)                                     │
│  ↓                                                          │
│  AliceBrainService.generate()                               │
│  ↓                                                          │
│  Build adapter stack (ENF + U/T/GU/GT + VOICE)            │
│  ↓                                                          │
│  Native Inference (iOS/Android)                            │
│  - Load base model + adapters                              │
│  - Generate raw text with structured tags                  │
│  ↓                                                          │
│  StructuredResponseParser.parse()                          │
│  - Extract <policy><actions><answer>                       │
│  - Repair if malformed                                     │
│  ↓                                                          │
│  GatingEngine.enforceGates()                                │
│  - Apply hard blocks (tier, agentic, domain, etc.)         │
│  - Filter blocked actions                                  │
│  ↓                                                          │
│  AnswerRepair.repair()                                      │
│  - Append domain-aware repair messages                    │
│  - Never leak system prompts                               │
│  ↓                                                          │
│  GatingEngine.enforceBrevity()                             │
│  - Limit length for live_workout                           │
│  ↓                                                          │
│  Return gated response to UI + Watch sync                  │
└─────────────────────────────────────────────────────────────┘
```

---

## Key Integration Points

### 1. Flutter Service Layer (`alice_brain_service.dart`)

**Before Native Call**:

- Build adapter stack via `MeshRouter`
- Apply memory safeguards (filter adapters if low RAM)
- Disable VOICE for `live_workout` domain

**After Native Call**:

```dart
// Parse structured response
final structuredResponse = StructuredResponseParser.parse(rawTextFromNative);

// Build capabilities
final caps = CapabilitiesFlags.fromContext(...);

// Apply gating
final gatingResult = GatingEngine.enforceGates(
  caps: caps,
  parsedPolicy: structuredResponse.policy,
  parsedActions: structuredResponse.actions,
  parsedAnswer: structuredResponse.answer,
  domain: request.context.domain.name,
);

// Enforce brevity
final finalAnswer = GatingEngine.enforceBrevity(
  gatingResult.repairedAnswer,
  request.context.domain.name,
  request.context.isActiveWorkout,
);
```

### 2. Native Layer (iOS/Android)

**Native layers remain thin**:

- Load base model + adapter stack
- Apply adapters in order (ENF first, VOICE last)
- Generate raw text with structured tags
- Return raw text to Flutter

**No gating logic in native** - all enforcement happens in Flutter for consistency.

### 3. Watch/Wearable Sync

**Uses post-gating answer**:

```dart
// Watch sync uses finalAnswer (already gated and repaired)
watchSyncService.sendPayload({
  'text': finalAnswer,  // Gated answer, not raw
  'actions': gatedActions,  // Filtered actions
});
```

---

## Hard Blocks Implementation

### Block 1: Free Tier - Plan Operations

```dart
// Free users CAN build plans manually in UI
// AI CANNOT produce executable plan-create actions
if (caps.tier == UserTier.free) {
  if (ActionSchema.isPlanOperation(actionType)) {
    return 'planOpsProOnly';  // Block
  }
}
```

**Result**: Free users get repair message: "I can't create plans automatically on your current tier, but you can still build a plan manually in the Plan Builder..."

### Block 2: Agentic Disabled

```dart
if (!caps.canUseAgenticActions) {
  if (requiresPro || schema?.requiresAgentic == true) {
    return 'agenticDisabled';  // Block
  }
}
```

### Block 3: Live Workout Restrictions

```dart
if (caps.isLiveWorkoutContext) {
  final allowed = {'none', 'adjust_live_rest', 'navigate'};
  if (!allowed.contains(actionType)) {
    return 'liveWorkoutRestricted';  // Block
  }
}
```

### Block 4: Domain Restrictions

```dart
if (schema != null && !schema.isAllowedInDomain(domain)) {
  return 'domainRestricted';  // Block
}
```

---

## Answer Repair Examples

### Free User Requests Plan Creation

**Model Output**:

```
<actions>{"actions":[{"type":"plan.create","payload":{...}}]}</actions>
<answer>I'll create a 4-day hypertrophy plan for you.</answer>
```

**After Gating**:

```
<actions>{"actions":[{"type":"none","payload":{},"requiresPro":false}]}</actions>
<answer>I'll create a 4-day hypertrophy plan for you.

I can't create plans automatically on your current tier, but you can still build a plan manually in the Plan Builder. If you want, I can guide you step-by-step. If you upgrade to Pro, I can generate the full plan for you.</answer>
```

### Live Workout - Action Blocked

**Model Output**:

```
<actions>{"actions":[{"type":"plan.create",...}]}</actions>
<answer>I'll create a plan for you after your workout.</answer>
```

**After Gating**:

```
<actions>{"actions":[{"type":"none",...}]}</actions>
<answer>I'll create a plan for you after your workout.

Actions limited during workout. Tell me your set.</answer>
```

---

## Memory Safeguards

### Runtime Adapter Loading

```dart
// Filter adapter stack based on constraints
final constraints = await MemorySafeguards.getResourceConstraints();
final filteredStack = MemorySafeguards.filterAdapterStack(
  adapterStack: adapterStack,
  constraints: constraints,
  domain: domain,
  isActiveWorkout: isActiveWorkout,
);

// VOICE disabled for live_workout
// Only ENF + U loaded if low RAM
```

### Training Safeguards

```dart
// Nightly training only runs if:
// - Device is charging
// - On WiFi
// - No resource constraints
final canTrain = MemorySafeguards.canRunNightlyTraining(constraints);
if (!canTrain) {
  // Skip training, try again tomorrow
  return;
}
```

---

## Testing Scenarios

### Test 1: Free User Plan Creation

- **Input**: Free user requests "Create me a 4-day plan"
- **Expected**: `plan.create` blocked, repair message added
- **Result**: ✅ Pass

### Test 2: Pro User Plan Creation

- **Input**: Pro user requests "Create me a 4-day plan"
- **Expected**: `plan.create` allowed
- **Result**: ✅ Pass

### Test 3: Live Workout Restrictions

- **Input**: User in live workout requests plan creation
- **Expected**: Action blocked, ultra-brief repair
- **Result**: ✅ Pass

### Test 4: Malformed Output

- **Input**: Model returns plain text without tags
- **Expected**: Parser repairs, uses safe defaults
- **Result**: ✅ Pass

### Test 5: Agentic Disabled

- **Input**: `agenticEnabled=false`, model proposes agentic action
- **Expected**: Action blocked, repair message
- **Result**: ✅ Pass

---

## Success Criteria

✅ **Free users cannot trigger AI plan creation actions**

- Hard block in code, not prompt
- Repair message provides manual alternative

✅ **Model can't "claim it executed" blocked actions**

- Actions filtered before execution
- Answer repaired deterministically
- No claims in final output

✅ **Output contract is stable**

- Parser handles malformed output
- Safe defaults always returned
- Never leaks system prompts

✅ **Identical behavior on iOS and Android**

- All gating logic in Flutter (Dart)
- Native layers only do inference
- Same rules, same results

---

## File Structure

```
flutter_app/lib/features/alice/domain/gating/
├── action_catalog.dart          # Action type constants
├── actions_schema.dart           # Comprehensive action definitions
├── capabilities_flags.dart       # Capability flags per request
├── gating_engine.dart            # Hard blocks and enforcement
├── answer_repair.dart            # Domain-aware repair messages
├── structured_response.dart      # Parser with repair pass
└── memory_safeguards.dart       # Resource constraints

flutter_app/test/gating/
└── gating_engine_test.dart       # Unit tests
```

---

## Integration Diff Summary

### Changes to `alice_brain_service.dart`

**Before**:

- Parsed uiSpec directly from native
- Applied basic gating
- Simple repair logic

**After**:

- Uses `StructuredResponseParser` for robust parsing
- Applies comprehensive `GatingEngine.enforceGates()`
- Uses `AnswerRepair` for domain-aware messages
- Enforces brevity for live workout
- All gating happens in Flutter (cross-platform consistency)

### Native Layer Changes

**No changes required** - native layers continue to:

- Load adapters
- Generate raw text
- Return to Flutter

All gating enforcement moved to Flutter for consistency.

---

## Key Design Principles

1. **Code over Prompts**: Hard blocks in Dart, not system prompts
2. **Deterministic Repair**: Same input → same repair message
3. **Never Leak**: Repair messages never expose system prompts
4. **Cross-Platform**: All logic in Flutter, native stays thin
5. **Defense in Depth**: ENF reduces violations, gating enforces, repair fixes

---

## Next Steps

1. ✅ Structured parser with repair
2. ✅ Comprehensive action schema
3. ✅ Hard blocks implementation
4. ✅ Domain-aware answer repair
5. ✅ Memory safeguards
6. ✅ Unit tests
7. ⏳ Integration testing on iOS/Android
8. ⏳ Performance validation
9. ⏳ Watch sync validation

## Related

^[source-materials/mirrors/doctrine/GATING_SYSTEM_INTEGRATION.md]
