---
title: MISSING_EXAMPLES_GENERATED
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/MISSING_EXAMPLES_GENERATED.md"]
updated: 2026-07-24
---

# Missing Auto Agentic Examples - Generated

## Summary

Generated **28 new training examples** covering all missing automatic agentic actions with proper trigger context.

## Generated Examples Breakdown

### 1. Strain Score-Based Rest Adjustments (12 examples)

**Category**: `strain_rest_*`
**Coverage**:

- High strain (>95%): 3 trigger variations
- Moderate strain (86-95%): 3 trigger variations
- At threshold (85%): 3 trigger variations
- Below threshold (<85%): 3 trigger variations

**Trigger Variations Include**:

- Simple: "Strain score 97% detected during rest period (above 95% threshold)"
- Detailed: "Current strain 97, baseline strain 45, recovery delay 38s. Strain calculation: (0.4 × 42) + (0.3 × 3) + (0.3 × 127) = 97%"
- Status-based: "Strain status: red (97%). System requesting automatic rest extension"

### 2. Extended Mesocycle Progressions (3 examples)

**Category**: `extended_mesocycle_*`
**Coverage**:

- Week 4 → Week 5 progression (6-week mesocycle)
- Week 5 → Week 6 progression (6-week mesocycle)
- 6-week mesocycle transition

**Triggers Include**:

- "User completed week 4 of 6-week mesocycle with 88% completion rate"
- "User completed week 5 of 6-week mesocycle with 92% completion rate"
- "User completed 6-week mesocycle"

### 3. Learning-Based Progression Selection (3 examples)

**Category**: `learning_progression_*`
**Coverage**:

- Choosing progression based on previous mesocycle results
- Coverage requirement (must try all types)
- Single-adjustment constraint

**Triggers Include**:

- "Previous mesocycle: add_set worked best (avg completion 92%)"
- "Must try all progression types. Already used: add_set, add_rep. Not yet used: increase_weight"
- "Single-adjustment rule: Can only use one progression type per week"

### 4. Alice's Perceived Intensity Score (3 examples)

**Category**: `alice_intensity_*`
**Coverage**:

- High intensity workout (85/100)
- Moderate intensity workout (55/100)
- Very high intensity workout (95/100)

**Triggers Include**:

- "Workout session completed. Total duration: 45 minutes. Average heart rate: 145 bpm. Total volume: 12,500 lbs. System requesting Alice's perceived intensity evaluation."

### 5. Music-Performance Correlation Analysis (2 examples)

**Category**: `music_correlation_*`
**Coverage**:

- General correlation analysis (35 sets)
- Exercise-specific correlation (45 sets, bench_press)

**Triggers Include**:

- "30+ sets observed (observedSetCount: 35), onboarding state: active. System requesting music-performance correlation analysis."
- "45 sets observed (observedSetCount: 45), onboarding state: active. System requesting music-performance correlation analysis for exercise: bench_press."

### 6. Performance-Based Playlist Creation (2 examples)

**Category**: `playlist_creation_*`
**Coverage**:

- Balancing user preferences with performance (32 sets)
- Prioritizing user preferences first, then performance (50 sets)

**Triggers Include**:

- "30+ sets observed (observedSetCount: 32), correlation patterns identified. User preferences: Track A (thumbsUp: 5, starred: true)..."
- "50 sets observed (observedSetCount: 50), correlation patterns identified. User preferences: Track D (thumbsUp: 2)..."

### 7. 6-Week Mesocycle Actions (3 examples)

**Category**: `six_week_*`
**Coverage**:

- 6-week mesocycle deload trigger
- 6-week exercise replacement (squat)
- 6-week exercise replacement (overhead_press)

**Triggers Include**:

- "Mesocycle complete (week 6/6). System requesting deload week."
- "Exercise completion 48% over 6-week mesocycle. System requesting exercise replacement."
- "Exercise completion 47% over 6-week mesocycle. System requesting exercise replacement."

## Total Coverage

| Category                       | Count  | Status                                       |
| ------------------------------ | ------ | -------------------------------------------- |
| Strain rest adjustments        | 12     | ✅ Complete with trigger variations          |
| Extended mesocycle (5-6 weeks) | 3      | ✅ Complete                                  |
| Learning-based progression     | 3      | ✅ Complete                                  |
| Alice's perceived intensity    | 3      | ✅ Complete (end of workout trigger)         |
| Music-performance correlation  | 2      | ✅ Complete (30+ sets requirement)           |
| Performance playlist creation  | 2      | ✅ Complete (30+ sets, respects preferences) |
| 6-week mesocycle actions       | 3      | ✅ Complete                                  |
| **TOTAL**                      | **28** | ✅ **All missing actions covered**           |

## Key Features

✅ **All triggers included**: Every example has proper SYSTEM CONTEXT with trigger conditions
✅ **Trigger variations**: Multiple ways to express the same trigger condition
✅ **Boundary conditions**: Examples at thresholds (85% strain, 50% completion)
✅ **6-week mesocycles**: All actions support both 4-week and 6-week mesocycles
✅ **Learning constraints**: Single-adjustment rule and coverage requirements
✅ **User preferences**: Music playlists respect user preferences first, then optimize

## Next Steps

Add to training data:

```bash
cd training/enf_lora
cat data/missing_auto_agentic_examples.jsonl >> data/enf_train.jsonl
```

**Total training examples after adding**:

- Existing: ~632 examples
- New: 28 examples
- **New total: ~660 examples**

## Related

^[{src_rel}]
