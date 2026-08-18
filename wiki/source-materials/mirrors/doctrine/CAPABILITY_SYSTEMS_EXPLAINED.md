---
title: CAPABILITY_SYSTEMS_EXPLAINED
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CAPABILITY_SYSTEMS_EXPLAINED.md"]
updated: 2026-07-24
---

# Two Capability Systems Explained

## Overview

There are **two different capability systems** that work together:

1. **CAPABILITIES_JSON** - Runtime permissions/config (already exists)
2. **alice_capability_map.json** - Action reference guide (what we're integrating)

---

## 1. CAPABILITIES_JSON (Runtime Config)

### What It Is

Compact JSON generated at runtime in `LlamaEngine.swift` that tells Alice what the user CAN do.

### Where It's Generated

```swift
// flutter_app/ios/Runner/LlamaEngine.swift
func promptHeader(domain: String, isActiveWorkout: Bool) -> String {
    let json = """
    {"tier":"\(effectiveTier.rawValue)","role":"\(role.rawValue)","appPhase":"\(appPhase.rawValue)","domain":"\(normalizedDomain)","agenticEnabled":\(agenticEnabled),"canUseAgenticActions":\(agenticEnabled),"canUseAiPlanOps":\(canUseAiPlanOps),"canUseNonFitnessHelp":\(canUseNonFitnessHelp),"isActiveWorkout":\(isActiveWorkout),"adminMode":\(isAdmin)}
    """
    return "INTERNAL_CONFIG:\(json)"
}
```

### Format

```json
INTERNAL_CONFIG:{"tier":"free","role":"user","appPhase":"beta","domain":"planning","agenticEnabled":false,"canUseAiPlanOps":false,"canUseAgenticActions":false,"canUseNonFitnessHelp":false,"isActiveWorkout":false,"adminMode":false}
```

### Purpose

- **Runtime permissions**: What the user CAN do right now
- **Access control**: Determines if actions are allowed
- **ENF Training**: ENF model is trained to follow this format

### Example

- `agenticEnabled: false` → User can't use agentic actions
- `canUseAiPlanOps: false` → User can't use AI plan operations
- `tier: "free"` → User is on free tier

---

## 2. alice_capability_map.json (Reference Guide)

### What It Is

Comprehensive capability reference that describes:

- What actions exist
- When to use each action
- How to use each action
- Payload structures
- Examples

### Location

`training/enf_lora/alice_capability_map.json`

### Format

```json
{
  "agenticActions": {
    "update_workout_plan": {
      "whatItDoes": "Creates or modifies workout plan...",
      "whenToUse": ["User requests new plan", "Week progression"],
      "access": {
        "requiresPro": true,
        "agenticEnabled": "must be true"
      },
      "payload": {
        "planId": "string (optional)",
        "changes": {...}
      },
      "howToUse": {
        "step1": "Check access rules...",
        "step2": "Construct payload..."
      }
    }
  }
}
```

### Purpose

- **Action reference**: What actions are available
- **Usage guide**: How to use each action
- **Payload structures**: What data to include
- **Training reference**: Can be used in training data

---

## How They Work Together

### Flow Example

**User**: "Create me a workout plan" (Free tier user)

1. **CAPABILITIES_JSON** (Runtime Config):

   ```json
   { "tier": "free", "agenticEnabled": false, "canUseAiPlanOps": false }
   ```

   → Tells Alice: "User is free tier, agentic disabled"

2. **alice_capability_map.json** (Reference Guide):

   ```json
   {
     "update_workout_plan": {
       "whatItDoes": "Creates workout plan",
       "access": { "requiresPro": true, "agenticEnabled": "must be true" }
     }
   }
   ```

   → Tells Alice: "update_workout_plan exists, requires Pro"

3. **Alice's Decision**:
   - Sees `update_workout_plan` in capability map
   - Checks CAPABILITIES_JSON: `agenticEnabled=false`, `tier=free`
   - Decides: "I can't use this action (blocked by runtime config)"
   - Generates answer: "I can't create plans automatically on your current tier..."

### Integration Strategy

**System Prompt Should Include Both**:

```
INTERNAL_CONFIG:{"tier":"free","agenticEnabled":false,...}

CAPABILITY_MAP:
{
  "update_workout_plan": {
    "whatItDoes": "...",
    "whenToUse": [...],
    "access": {"requiresPro": true},
    "payload": {...}
  },
  ...
}

Output format: <policy>...</policy><actions>...</actions><answer>...</answer>
```

**Alice Uses Both**:

- **CAPABILITIES_JSON**: Check permissions (can I do this?)
- **Capability Map**: Find actions (what actions exist? how do I use them?)

---

## ENF Training

### CAPABILITIES_JSON

- ✅ **Already trained**: ENF is trained to follow this format
- ✅ **In system prompt**: Injected as `INTERNAL_CONFIG:`
- ✅ **Runtime generated**: Created dynamically based on user context

### alice_capability_map.json

- ⚠️ **Not yet integrated**: Should be added to system prompt
- ⚠️ **Training needed**: ENF should learn to reference this for action selection
- ⚠️ **Filtered**: Only include relevant capabilities (domain, tier, role)

---

## Implementation Plan

### Phase 1: Load and Filter

1. Load `alice_capability_map.json` in `LlamaEngine.swift`
2. Filter by context (domain, tier, role)
3. Create compact format (remove examples, keep structure)

### Phase 2: Inject into Prompt

1. Add filtered capability map to system prompt
2. Place after `CAPABILITIES_JSON`
3. Keep token usage low (<500 tokens)

### Phase 3: Training Integration

1. Add examples showing Alice using capability map
2. Train ENF to reference map for action selection
3. Train ENF to combine with CAPABILITIES_JSON for decisions

---

## Key Differences

| Aspect           | CAPABILITIES_JSON   | alice_capability_map.json       |
| ---------------- | ------------------- | ------------------------------- |
| **Format**       | Compact JSON        | Detailed JSON with instructions |
| **Size**         | ~100-200 tokens     | ~200-500 tokens (filtered)      |
| **Purpose**      | Permissions/config  | Action reference                |
| **Generated**    | Runtime (dynamic)   | Static file (loaded)            |
| **ENF Training** | ✅ Already trained  | ⚠️ Needs integration            |
| **Location**     | `LlamaEngine.swift` | `training/enf_lora/`            |

---

## Summary

- **CAPABILITIES_JSON**: "What CAN the user do?" (permissions)
- **alice_capability_map.json**: "What actions exist and how do I use them?" (reference)

**Both should be in the system prompt**:

- CAPABILITIES_JSON for permissions
- Capability map for action details

**ENF uses both**:

- Checks CAPABILITIES_JSON for permissions
- References capability map for action details
- Combines both to make decisions

## Related

^[{src_rel}]
