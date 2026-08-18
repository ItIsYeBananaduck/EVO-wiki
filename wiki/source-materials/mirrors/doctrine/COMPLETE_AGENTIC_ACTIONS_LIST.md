---
title: COMPLETE_AGENTIC_ACTIONS_LIST
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/COMPLETE_AGENTIC_ACTIONS_LIST.md"]
updated: 2026-07-24
---

# Complete List of All Agentic Actions

## ✅ All Actions Covered in Training Dataset

### User-Requested Actions

| Action Type                              | Examples                                 | Status | File                                 | Count |
| ---------------------------------------- | ---------------------------------------- | ------ | ------------------------------------ | ----- |
| **navigate**                             | Navigate to workouts, nutrition, profile | ✅     | comprehensive_agentic_examples.jsonl | 48    |
| **update_workout_plan** (CREATE)         | "Create a push pull leg plan"            | ✅     | comprehensive_agentic_examples.jsonl | ~45   |
| **update_workout_plan** (UPDATE)         | "Update my workout plan"                 | ✅     | comprehensive_agentic_examples.jsonl | ~48   |
| **update_nutrition_targets**             | "Set my protein to 150g"                 | ✅     | comprehensive_agentic_examples.jsonl | 57    |
| **adjust_live_rest** (user-requested)    | "I need longer rest"                     | ✅     | comprehensive_agentic_examples.jsonl | 17    |
| **enforce_deload_stop** (user-expressed) | "I've been training 8 weeks"             | ✅     | comprehensive_agentic_examples.jsonl | 30    |
| **schedule_mesocycle_update**            | "Schedule a program update"              | ✅     | comprehensive_agentic_examples.jsonl | 30    |
| **update_profile**                       | "Update my weight to 75kg"               | ✅     | comprehensive_agentic_examples.jsonl | 36    |

### Automatic System-Triggered Actions

| Action Type                                    | Trigger                             | Status | File                                                                | Count |
| ---------------------------------------------- | ----------------------------------- | ------ | ------------------------------------------------------------------- | ----- |
| **adjust_live_rest** (HR-based)                | HR >88% or <68% of max              | ✅     | all_auto_agentic_examples.jsonl                                     | 3     |
| **adjust_live_rest** (RPE-based)               | Perceived exertion ≥9               | ✅     | all_auto_agentic_examples.jsonl                                     | 1     |
| **update_workout_plan** (week progression)     | Week 1→2, 2→3, 3→4                  | ✅     | incremental_update_examples.jsonl + all_auto_agentic_examples.jsonl | 7     |
| **update_workout_plan** (mesocycle)            | 4-week cycle complete               | ✅     | incremental_update_examples.jsonl                                   | 1     |
| **update_workout_plan** (volume adjustment)    | Post-session volume delta           | ✅     | all_auto_agentic_examples.jsonl                                     | 3     |
| **update_workout_plan** (exercise replacement) | Completion <50% over 4 weeks        | ✅     | all_auto_agentic_examples.jsonl                                     | 2     |
| **update_workout_plan** (progression engine)   | Completion/effort/recovery analysis | ✅     | all_auto_agentic_examples.jsonl                                     | 4     |
| **enforce_deload_stop** (readiness)            | Readiness score ≤45                 | ✅     | all_auto_agentic_examples.jsonl                                     | 1     |
| **enforce_deload_stop** (mesocycle)            | Mesocycle complete                  | ✅     | all_auto_agentic_examples.jsonl                                     | 1     |
| **enforce_deload_stop** (weeks)                | 6+ weeks without deload             | ✅     | all_auto_agentic_examples.jsonl                                     | 1     |
| **schedule_mesocycle_update** (deload)         | Mesocycle complete, schedule deload | ✅     | all_auto_agentic_examples.jsonl                                     | 1     |

## Training Dataset Files

1. **comprehensive_agentic_examples.jsonl** (557 examples)
   - All user-requested actions
   - All action types with multiple phrasings
   - Pro and free tier scenarios

2. **incremental_update_examples.jsonl** (4 examples)
   - Week-over-week progressions
   - Mesocycle transitions

3. **all_auto_agentic_examples.jsonl** (18 examples)
   - System-triggered rest adjustments
   - Volume adjustments
   - System-triggered deloads
   - Exercise replacements
   - Progression engine decisions
   - Week progression variations

4. **admin_debugging_examples.jsonl** (25 examples)
   - Admin technical questions
   - Capability listings
   - Architecture explanations

5. **agentic_action_examples.jsonl** (28 examples)
   - Direct action requests
   - "Create/Build/Design" patterns

**Total Training Examples: ~632 examples**

## Action Details

### 1. Dynamic Rest Adjustments (AUTO)

- **Type**: `adjust_live_rest`
- **Auto-Applied**: Yes (always)
- **Triggers**:
  - HR >88% max → Extend rest +15s
  - HR <68% max → Shorten rest -15s
  - RPE ≥9 → Extend rest +15s
- **Coverage**: ✅ 4 examples (3 HR-based, 1 RPE-based)

### 2. Week-Over-Week Progressions (AUTO)

- **Type**: `update_workout_plan` with progression
- **Auto-Applied**: Yes (in 'auto' mode)
- **Progression Types**:
  - Week 1→2: `add_set` (add 1 set)
  - Week 2→3: `add_rep` (add 1 rep)
  - Week 3→4: `increase_weight` (2.5-5lbs)
- **Coverage**: ✅ 7 examples (4 base + 3 variations)

### 3. Mesocycle Transitions (AUTO)

- **Type**: `update_workout_plan` with mesocycle
- **Auto-Applied**: Yes (in 'auto' mode)
- **Trigger**: After 4-week mesocycle complete
- **Coverage**: ✅ 1 example

### 4. Volume Adjustments (AUTO)

- **Type**: `update_workout_plan` with volume changes
- **Auto-Applied**: Yes (in 'auto' mode)
- **Triggers**:
  - Volume increase >15% → Cap at 15%
  - Volume decrease >40% → Cap at 40%
  - Low RPE (≤7) + small change → Gently increase
- **Coverage**: ✅ 3 examples

### 5. Deload Decisions (AUTO)

- **Type**: `enforce_deload_stop` or `schedule_mesocycle_update`
- **Auto-Applied**: Yes (in 'auto' mode)
- **Triggers**:
  - Readiness score ≤45
  - Mesocycle complete (week 4/4)
  - 6+ weeks without deload
- **Coverage**: ✅ 3 examples (system-triggered) + 30 examples (user-expressed)

### 6. Exercise Replacements (AUTO)

- **Type**: `update_workout_plan` with exerciseReplacement
- **Auto-Applied**: Yes (in 'auto' mode)
- **Trigger**: Exercise completion <50% over 4 weeks
- **Coverage**: ✅ 2 examples

### 7. Progression Engine Decisions (AUTO)

- **Type**: `update_workout_plan` with progression
- **Auto-Applied**: Yes (in 'auto' mode)
- **Decision Types**:
  - High completion (≥90%) + declining effort → Increase load 2.5%
  - Very high completion (≥95%) + stable recovery → Increase load 2%
  - Good completion (≥85%) + high effort → Add rep
  - Poor completion (<75%) or declining recovery → Deload 5%
- **Coverage**: ✅ 4 examples

## Summary

✅ **ALL automatic agentic actions are now covered in the training dataset!**

- User-requested actions: 557 examples
- Automatic system-triggered actions: 22 examples
- Admin debugging: 25 examples
- Direct action requests: 28 examples

**Total: ~632 training examples covering all agentic capabilities**

## Related

^[{src_rel}]
