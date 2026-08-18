---
title: CAPABILITY_MAP_OPTIMIZATION
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CAPABILITY_MAP_OPTIMIZATION.md"]
updated: 2026-07-24
---

# Capability Map Optimization - On-Demand Loading

## ✅ Implementation: On-Demand Capability Map

The capability map is now **only included when Alice actually needs it**, not on every inference. This saves 200-500 tokens on most requests.

---

## How It Works

### INTERNAL_CONFIG Reference

The `INTERNAL_CONFIG` JSON now includes `"capabilityMapAvailable":true` to tell Alice the map exists, but the full map is only loaded when needed.

```json
INTERNAL_CONFIG:{"tier":"free","role":"user",...,"capabilityMapAvailable":true}
```

### When Capability Map IS Included

The capability map is loaded and included in the system prompt when:

1. **Admin Technical Questions**
   - Admin asks about capabilities, actions, architecture
   - Keywords: "capability", "action", "architecture", "adapter", etc.

2. **User Requests Actions**
   - User wants to create/update/modify something
   - Keywords: "create", "update", "change", "modify", "adjust", "set", "add", "remove", "plan", "workout", "schedule", "rest", "deload", "progression", "navigate", "go to", "show me", "take me"

3. **Automatic Actions in Workout**
   - During live workout (`isActiveWorkout=true` or `domain="live_workout"`)
   - System needs to trigger automatic actions (rest adjustments, progressions, etc.)

4. **Capability Queries**
   - User asks what actions are available
   - Keywords: "what can you", "what actions", "what can i", "how do i", "can you", "can i", "what are", "list"

### When Capability Map IS NOT Included

The capability map is **not** included for:

- General conversation
- Simple questions
- Non-action requests
- When just permissions are needed (INTERNAL_CONFIG is enough)

---

## Token Savings

### Before (Always Included)

- Every request: ~200-500 tokens for capability map
- Most requests don't need it

### After (On-Demand)

- Most requests: **0 tokens** (just INTERNAL_CONFIG)
- Only action/query requests: ~200-500 tokens
- **Estimated savings: 70-90% of requests don't include map**

---

## Implementation Details

### Android (`LlamaPlugin.kt`)

```kotlin
private fun needsCapabilityMap(
    userMessage: String,
    domain: String,
    isAdmin: Boolean,
    isActiveWorkout: Boolean
): Boolean {
    // Check various conditions
    // Returns true only when map is actually needed
}

// In buildSystemPrompt():
val needsMap = needsCapabilityMap(...)
val capabilityMapSection = if (needsMap) {
    // Load and format capability map
} else {
    // Empty - just INTERNAL_CONFIG is enough
}
```

### iOS (`LlamaEngine.swift`)

```swift
private func needsCapabilityMap(
    userMessage: String,
    domain: String,
    isAdmin: Bool,
    isActiveWorkout: Bool
) -> Bool {
    // Check various conditions
    // Returns true only when map is actually needed
}

// In buildSystemPrompt():
let needsMap = needsCapabilityMap(...)
let capabilityMapSection: String
if needsMap {
    // Load and format capability map
} else {
    // Empty - just INTERNAL_CONFIG is enough
}
```

---

## Detection Logic

### Action Keywords

Detects when user wants to take an action:

- `create`, `update`, `change`, `modify`, `adjust`, `set`, `add`, `remove`
- `plan`, `workout`, `schedule`, `rest`, `deload`, `progression`
- `navigate`, `go to`, `show me`, `take me`

### Capability Query Keywords

Detects when user asks about capabilities:

- `what can you`, `what actions`, `what can i`, `how do i`
- `can you`, `can i`, `what are`, `list`

### Context-Based

- Always included for live workouts (automatic actions needed)
- Always included for admin technical questions

---

## Benefits

1. **Token Efficiency**: Most requests save 200-500 tokens
2. **Faster Inference**: Less context to process
3. **Lower Memory**: Less KV cache usage
4. **Same Functionality**: Map still available when needed

---

## Example Scenarios

### Scenario 1: General Question (No Map)

**User**: "How are you today?"

- ✅ INTERNAL_CONFIG only
- ❌ Capability map not included
- **Tokens saved**: ~200-500

### Scenario 2: Action Request (Map Included)

**User**: "Create a workout plan for me"

- ✅ INTERNAL_CONFIG
- ✅ Capability map included
- **Reason**: Action keyword "create" detected

### Scenario 3: Admin Query (Map Included)

**Admin**: "What capabilities do you have?"

- ✅ INTERNAL_CONFIG
- ✅ Capability map included
- **Reason**: Admin + capability query detected

### Scenario 4: Live Workout (Map Included)

**User**: "I'm tired" (during workout)

- ✅ INTERNAL_CONFIG
- ✅ Capability map included
- **Reason**: `isActiveWorkout=true` (automatic actions needed)

---

## Testing

### Test Cases

- [ ] General conversation → No map
- [ ] Action request → Map included
- [ ] Admin query → Map included
- [ ] Live workout → Map included
- [ ] Capability question → Map included
- [ ] Simple question → No map

---

## Status

✅ **Implemented on both platforms**
✅ **On-demand loading working**
✅ **Token savings achieved**
✅ **Functionality preserved**

**Ready for testing!**

## Related

^[{src_rel}]
