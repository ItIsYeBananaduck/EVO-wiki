---
title: CAPABILITY_MAP_USAGE
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/CAPABILITY_MAP_USAGE.md
updated: 2026-07-24
---

# Using Alice Capability Map

## Overview

The `alice_capability_map.json` is a comprehensive reference guide that Alice can consult when performing agentic tasks. Instead of relying solely on training data, Alice can look up:

- What actions are available
- When to use each action
- What triggers each automatic action
- Access rules and constraints
- Payload structures

## How to Use in System Context

Include the capability map in Alice's system context when she needs to perform agentic tasks:

```json
{
  "role": "system",
  "content": "CONTEXT:
- Tier: pro
- Role: user (isAdmin: false, isTrainer: false, isUser: true)
- AgenticEnabled: true
- AppPhase: production

You are Alice, an AI fitness coach...

CAPABILITY MAP: [Include relevant sections from alice_capability_map.json]

When an agentic task needs to be performed:
1. Consult the capability map to understand available actions
2. Check access rules (requiresPro, agenticEnabled)
3. Identify the appropriate action type
4. Construct the payload according to the map
5. Generate the action with proper format

SYSTEM CONTEXT: [Trigger context if automatic action]"
}
```

## Key Sections

### 1. Agentic Actions

All user-requested and system-triggered actions:

- `navigate` - Screen navigation
- `update_workout_plan` - Create/update workout plans
- `update_nutrition_targets` - Set nutrition goals
- `adjust_live_rest` - Rest time adjustments
- `enforce_deload_stop` - Safety deloads
- `schedule_mesocycle_update` - Schedule updates
- `update_profile` - Profile changes
- `none` - No action needed

### 2. Automatic Actions

System-triggered actions with specific triggers:

- Strain-based rest adjustments (PRIMARY)
- Week-over-week progressions
- Mesocycle transitions
- Exercise replacements
- Volume adjustments
- System deloads
- Alice's perceived intensity
- Music-performance correlation
- Performance playlist creation

### 3. Access Rules

- `agenticEnabled` - Primary gate
- `requiresPro` - Pro-tier requirement
- `domainConstraints` - Context-specific rules

### 4. Execution Rules

- Auto-execute vs require confirmation
- Trainer approval requirements

## Example Usage

### User Request: "Create a push/pull/legs plan"

Alice consults the map:

1. **Action Type**: `update_workout_plan`
2. **Create vs Update**: User said "create" → omit `planId`
3. **Access Check**: `requiresPro: true` → check `agenticEnabled: true` ✅
4. **Payload**:
   ```json
   {
     "changes": {
       "schedule": {
         "type": "push_pull_legs",
         "daysPerWeek": 6
       },
       "notes": "User requested new push/pull/legs plan"
     }
   }
   ```
5. **Confirmation**: `requiresConfirmation: true` → inform user

### System Trigger: "Strain score 97% detected during rest"

Alice consults the map:

1. **Action Type**: `adjust_live_rest` (from automaticActions.strainBasedRestAdjustment)
2. **Trigger Match**: Strain >95% (red zone)
3. **Access Check**: `requiresPro: false`, `autoExecute: true` ✅
4. **Payload**:
   ```json
   {
     "restSeconds": 120,
     "reason": "Strain score 97% (red zone) - extending rest significantly",
     "strainScore": 97,
     "strainStatus": "red"
   }
   ```
5. **Execution**: Auto-execute (no confirmation needed)

## Benefits

1. **Explicit Knowledge**: Alice doesn't need to learn all rules from training data
2. **Up-to-date**: Map can be updated without retraining
3. **Consistency**: All actions follow the same structure
4. **Trigger Recognition**: Clear triggers for automatic actions
5. **Access Control**: Explicit access rules prevent violations

## Integration

### Option 1: Include in System Prompt

Add relevant sections to Alice's system context based on current situation.

### Option 2: Reference on Demand

When Alice needs to perform an action, she can reference the map:

- "Consulting capability map for update_workout_plan..."
- "Checking access rules for adjust_live_rest..."

### Option 3: Embed in Training

Include the map structure in training examples so Alice learns to reference it.

## Next Steps

1. **Test the map** with sample queries
2. **Refine triggers** based on actual system behavior
3. **Add examples** for edge cases
4. **Update automatically** as new actions are added
5. **Version control** the map alongside training data

## Related

^[source-materials/mirrors/doctrine/CAPABILITY_MAP_USAGE.md]
