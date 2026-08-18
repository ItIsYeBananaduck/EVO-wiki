---
title: CAPABILITY_MAP_INFERENCE_OPTIMIZATION
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CAPABILITY_MAP_INFERENCE_OPTIMIZATION.md"]
updated: 2026-07-24
---

# Capability Map Inference Optimization

## ✅ Yes - Inference Stays Lite When Map Not Needed

The implementation is optimized to keep inference lightweight when the capability map isn't needed.

---

## Token Usage Breakdown

### When Capability Map IS Needed (Action/Query Requests)

**System Prompt Includes:**

```
INTERNAL_CONFIG:{"tier":"free",...,"capabilityMapAvailable":true}  (~100 tokens)

CAPABILITY_MAP:
{filtered and compacted map}  (~200-500 tokens)

CAPABILITY_MAP USAGE:
- Instructions on how to use the map  (~50 tokens)

Output format: <policy>...</policy><actions>...</actions><answer>...</answer>  (~20 tokens)

RULES: ...  (~30 tokens)
```

**Total**: ~400-700 tokens

### When Capability Map IS NOT Needed (General Conversation)

**System Prompt Includes:**

```
INTERNAL_CONFIG:{"tier":"free",...,"capabilityMapAvailable":true}  (~100 tokens)

Output format: <policy>...</policy><actions>...</actions><answer>...</answer>  (~20 tokens)

RULES: ...  (~30 tokens)
```

**Total**: ~150 tokens

**Savings**: ~250-550 tokens per request (70-90% of requests)

---

## Optimization Details

### 1. Early Detection

```kotlin
// Check BEFORE loading anything
val needsMap = needsCapabilityMap(userMessage, domain, isAdmin, isActiveWorkout)
```

**Benefit**: No file I/O or parsing if not needed

### 2. Conditional Loading

```kotlin
val capabilityMapSection = if (needsMap) {
    // Only load if needed
    loadRelevantCapabilities(...)
} else {
    ""  // Empty string - no tokens
}
```

**Benefit**: File is never opened when not needed

### 3. No Instructions When Not Needed

```kotlin
val capabilityMapInstructions = if (needsMap && capabilityMapSection.isNotEmpty()) {
    // Instructions only when map is present
    "CAPABILITY_MAP USAGE: ..."
} else {
    ""  // No instructions - saves ~50 tokens
}
```

**Benefit**: No mention of capability map when it's not included

### 4. INTERNAL_CONFIG Only Reference

```json
{ "capabilityMapAvailable": true }
```

**Benefit**: Alice knows map exists, but doesn't need details for general conversation

---

## Performance Impact

### When Map Not Needed (Most Requests)

**Before Optimization:**

- Always loaded map: ~200-500 tokens
- Always included instructions: ~50 tokens
- **Total overhead**: ~250-550 tokens on every request

**After Optimization:**

- Map not loaded: 0 tokens
- No instructions: 0 tokens
- **Total overhead**: 0 tokens

**Improvement**: **100% reduction** in capability map overhead for general conversation

### When Map IS Needed (Action/Query Requests)

**Token Cost:**

- Map content: ~200-500 tokens (necessary)
- Instructions: ~50 tokens (necessary)
- **Total**: ~250-550 tokens (acceptable, only when needed)

---

## Detection Logic

The `needsCapabilityMap()` function detects when map is needed:

### ✅ Map IS Included When:

1. **Admin technical questions** - "What capabilities do you have?"
2. **Action requests** - "Create a workout plan", "Update my schedule"
3. **Live workout context** - Automatic actions needed
4. **Capability queries** - "What can you do?", "List actions"

### ❌ Map IS NOT Included When:

1. **General conversation** - "How are you?", "Tell me about fitness"
2. **Simple questions** - "What's a good workout?"
3. **Non-action requests** - "Explain progressive overload"
4. **Permission checks only** - INTERNAL_CONFIG is sufficient

---

## Verification

### Android

```kotlin
// Early check - no loading if not needed
val needsMap = needsCapabilityMap(...)  // ~0.1ms (string matching)

if (needsMap) {
    // Only executed when needed
    loadRelevantCapabilities(...)  // ~5-10ms (file I/O + parsing)
    formatCapabilityMap(...)  // ~1-2ms (formatting)
}
// Otherwise: 0ms overhead
```

### iOS

```swift
// Early check - no loading if not needed
let needsMap = needsCapabilityMap(...)  // ~0.1ms (string matching)

if needsMap {
    // Only executed when needed
    loadRelevantCapabilities(...)  // ~5-10ms (file I/O + parsing)
    formatCapabilityMap(...)  // ~1-2ms (formatting)
}
// Otherwise: 0ms overhead
```

---

## Summary

✅ **Inference stays lite when map not needed:**

- No file I/O
- No JSON parsing
- No formatting
- No instructions
- **0 tokens added**

✅ **Map only included when needed:**

- Action requests
- Admin queries
- Live workout context
- Capability questions

✅ **Token savings:**

- Most requests: **0 tokens** (vs 250-550 before)
- Action requests: ~250-550 tokens (acceptable, necessary)

**Result**: Inference is optimized - lightweight for general conversation, detailed when actions are needed.

## Related

^[{src_rel}]
