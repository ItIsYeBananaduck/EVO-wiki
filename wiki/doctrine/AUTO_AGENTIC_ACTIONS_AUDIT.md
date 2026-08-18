---
title: AUTO_AGENTIC_ACTIONS_AUDIT
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/AUTO_AGENTIC_ACTIONS_AUDIT.md"]
updated: 2026-07-24
---

# Auto Agentic Actions Audit

This document lists ALL automatic agentic actions Alice can perform and whether they're covered in the training dataset.

## Automatic Agentic Actions

### ✅ 1. Dynamic Rest Adjustments (LIVE WORKOUT)

**Action Type**: `adjust_live_rest`
**Trigger**: During live workout, based on **strain score** (PRIMARY - HR is redundant since strain already includes HR)
**Auto-Applied**: Yes (always auto, no confirmation)
**Status**: ⚠️ **PARTIALLY IN DATASET**

- **Location**: `comprehensive_agentic_examples.jsonl` (17 examples)
- **Examples**: "I need longer rest", "Give me more rest time", "Increase my rest period"
- **Coverage**: User-requested rest adjustments covered
- **Missing**: System-triggered automatic rest adjustments based on **strain score**

**Important**: Strain score formula is `(0.4 × HR rise) + (0.3 × SpO₂ drop) + (0.3 × recovery delay)`. Since HR is already part of strain calculation, using HR separately for rest adjustments is **redundant**. Use strain score directly.

**Details**:

- **Strain score-based adjustments** (PRIMARY - use this, not HR):
  - Strain >85% (yellow/red) → Auto-extend rest
  - Strain ≤85% (green) → Standard rest
  - High strain (>95%) → Extend rest significantly
- **RPE-based adjustments** (secondary):
  - High RPE (≥9) → Extend rest by 15s
- Applied automatically during live workouts

---

### ✅ 2. Week-Over-Week Progressions (AUTOMATIC)

**Action Type**: `update_workout_plan` with progression changes
**Trigger**: After completing each week of mesocycle
**Auto-Applied**: Yes (in 'auto' autonomy mode)
**Status**: ⚠️ **PARTIALLY IN DATASET**

**Important Constraints**:

- **Mesocycle length**: Can be up to **6 weeks** (not just 4)
- **Learning requirement**: Alice must learn which adjustments work best for each user
- **Single adjustment rule**: Can adjust any progression type, but **only one at a time** (cannot combine multiple adjustments)
- **Coverage requirement**: Must try all progression types **at least once** over time

**Progression Types** (Alice can choose any, but only one per week):

- **`add_set`**: Add 1 set per exercise
- **`add_rep`**: Add 1 rep per set
- **`increase_weight`**: Increase weight by 2.5-5lbs
- **`add_volume`**: Increase total volume (alternative to weight increase)
- **`replace_exercise`**: Replace exercise with alternative (if completion <50%)

**Week Progression Examples**:

- Week 1 → Week 2: `add_set` (add 1 set)
- Week 2 → Week 3: `add_rep` (add 1 rep)
- Week 3 → Week 4: `increase_weight` (2.5-5lbs)
- Week 4 → Week 5: `add_volume` (if mesocycle extends to 5-6 weeks)
- Week 5 → Week 6: Different progression based on what works best

**Dataset Coverage**:

- ✅ **Location**: `incremental_update_examples.jsonl` (4 examples)
- ✅ Covers: Week 1→2, Week 2→3, Week 3→4, Mesocycle transition
- ❌ **Missing**:
  - 5-6 week mesocycle progressions
  - Examples showing Alice learning/choosing which progression type works best
  - Examples showing single-adjustment constraint (only one type per week)
  - Examples showing all progression types being tried over time

---

### ✅ 3. Mesocycle Transitions (AUTOMATIC)

**Action Type**: `update_workout_plan` with mesocycle changes
**Trigger**: After completing mesocycle (4-6 weeks)
**Auto-Applied**: Yes (in 'auto' autonomy mode)
**Status**: ⚠️ **PARTIALLY IN DATASET**

- **Location**: `incremental_update_examples.jsonl` (1 example)
- **Coverage**: Mesocycle 1 → Mesocycle 2 transition (4-week example)
- ❌ **Missing**:
  - 5-6 week mesocycle transitions
  - Different progression type transitions (set→rep→volume cycles)
  - Examples showing Alice choosing next mesocycle progression based on what worked best

**Details**:

- Mesocycle length: **4-6 weeks** (not fixed at 4)
- Cycles through progression types: `add_set` → `add_rep` → `add_volume` → repeat
- Each mesocycle uses different progression type
- Alice learns which progression types work best and can adjust mesocycle length accordingly
- Automatically starts new mesocycle with different progression focus

---

### ⚠️ 4. Volume Adjustments (POST-SESSION)

**Action Type**: `update_workout_plan` with volume changes
**Trigger**: After workout session, based on volume delta vs previous session
**Auto-Applied**: Yes (in 'auto' autonomy mode)
**Status**: ❌ **NOT IN DATASET**

**Details**:

- Compares current session volume vs previous session
- If volume increase >15% → Cap at 15% (guardrail)
- If volume decrease >40% → Cap at 40% (guardrail)
- If low RPE (≤7) and small change (<2%) → Gently increase volume
- Applied automatically in 'auto' mode, suggested in 'co-author' mode

**Missing Examples**:

- "Session volume increased 20%, capping at 15% per guardrail"
- "Low RPE (6) detected, gently increasing volume"
- "Volume decreased 50%, capping at 40% per guardrail"

---

### ✅ 5. Deload Decisions (AUTOMATIC)

**Action Type**: `enforce_deload_stop` or `schedule_mesocycle_update`
**Trigger**:

- When mesocycle complete (week 4/4 **OR week 6/6** - mesocycle length varies)
- When readiness score ≤45
- When 6+ weeks of training without deload
  **Auto-Applied**: Yes (in 'auto' autonomy mode)
  **Status**: ⚠️ **PARTIALLY IN DATASET**

**Dataset Coverage**:

- ✅ **Location**: `comprehensive_agentic_examples.jsonl` (30 examples)
- ✅ Covers: User expressing exhaustion, overtraining
- ❌ **Missing**: System-triggered deloads (readiness score, mesocycle complete)

**Missing Examples** (need both 4-week and 6-week mesocycle variations):

- "Readiness score 42 below threshold 45, scheduling deload"
- "Mesocycle complete (week 4/4), scheduling deload week"
- "Mesocycle complete (week 6/6), scheduling deload week"
- "6 weeks of training without deload, enforcing rest"

---

### ⚠️ 6. Exercise Replacements (AUTOMATIC)

**Action Type**: `update_workout_plan` with exercise swap
**Trigger**: When exercise completion <50% over mesocycle (4-week **OR 6-week** mesocycle)
**Auto-Applied**: Yes (in 'auto' autonomy mode)
**Status**: ❌ **NOT IN DATASET**

**Details**:

- Tracks completion rates for each exercise over the mesocycle (4 or 6 weeks)
- If average completion <50% → Replace exercise
- Finds replacement from same muscle group
- Applied automatically in 'auto' mode

**Missing Examples** (need both 4-week and 6-week mesocycle variations):

- "Squat completion 45% over 4 weeks, replacing with leg press"
- "Squat completion 48% over 6 weeks, replacing with leg press"
- "Exercise swap: bench press → dumbbell press (low completion over 4 weeks)"
- "Exercise swap: overhead press → dumbbell shoulder press (low completion over 6 weeks)"

---

### ⚠️ 7. Progression Engine Decisions (AUTOMATIC)

**Action Type**: `update_workout_plan` with progression changes
**Trigger**: Based on completion rate, effort trend, recovery
**Auto-Applied**: Yes (in 'auto' autonomy mode)
**Status**: ❌ **NOT IN DATASET**

**Decision Types**:

- **High completion (≥90%) + declining effort** → Increase load 2.5% OR add rep
- **Very high completion (≥95%) + stable recovery** → Increase load 2%
- **Good completion (≥85%) + high effort** → Add rep (volume progression)
- **Poor completion (<75%) OR declining recovery** → Deload 5%

**Missing Examples**:

- "Completion 92% with declining effort, increasing load 2.5%"
- "Completion 96% with stable recovery, conservative 2% increase"
- "Completion 78% with recovery decline, reducing load 5%"

---

### ⚠️ 8. Heart Rate-Based Adjustments (LIVE WORKOUT) - **REDUNDANT**

**Action Type**: `adjust_live_rest` (automatic)
**Status**: ❌ **REDUNDANT - Same as #1 (Dynamic Rest Adjustments)**

**Note**: This is the **same as Dynamic Rest Adjustments (#1)**. Since strain score already includes HR in its formula `(0.4 × HR rise) + (0.3 × SpO₂ drop) + (0.3 × recovery delay)`, using HR separately is redundant. Use strain score directly for rest adjustments.

**Do NOT create separate training examples for this** - it's already covered by strain-based rest adjustments.

---

## ⚠️ NOT Agentic Actions (But Related Features)

### StrainSync (Music Playlist Curation)

**Status**: ❌ **NOT an agentic action** (but Alice SHOULD analyze music-performance patterns)
**What it is**: System service for music playlist curation based on workout intensity/strain
**Current Implementation**:

- Uses strain/intensity scores to select music tempo bands (recovery/steady/push)
- Curates playlists from Spotify/Apple Music based on workout intensity
- Backend API endpoint: `/strain-sync` (calculates strain from songs + HR)
- **Current limitation**: Only scores tracks based on user preferences (thumbs up/down, play count)

**Missing Feature - Alice Should Do This Agentically**:

- ❌ **Alice should analyze music-performance correlation patterns set-over-set**
- ❌ **Alice should identify which tracks correlate with better performance** (higher reps, lower RPE, better form)
- ❌ **Alice should create playlists based on performance correlation**, while **respecting user's musical preferences**
- ❌ **Training data needed**: Examples showing Alice analyzing set-level music-performance patterns and creating performance-optimized playlists that balance performance data with user preferences

**Requirements**:

- **Minimum threshold**: Alice needs **at least 30 sets** of observed data (`observedSetCount >= 30`) before she can use music-performance correlation
- **Onboarding state**: Starts as `manual`, transitions to `active` after 30 sets
- **Balance**: Alice must balance user musical preferences (thumbs up/down, starred, play count) with performance correlation data
- **User preferences are primary**: Alice should respect what the user likes/dislikes, then optimize within those preferences based on performance

**Data Available** (per `STRAINSYNC_TRAINING_DATA.md`):

- Set-level music logs: track played, performance metrics (RPE, reps, weight, HR) for that set
- User preferences: `StrainSyncTrackMeta` with `thumbsUpCount`, `thumbsDownCount`, `skipCount`, `isStarred`, `playCount`
- This data exists but is NOT being used for correlation analysis or performance-based playlist creation

---

## ⚠️ Missing Features (Not in Code Yet)

### 1. Alice's Perceived Intensity Score

**Status**: ❌ **NOT IMPLEMENTED**
**What it should be**: Alice should provide her perceived intensity score that gets blended with mechanical intensity to calculate the "true score"
**Trigger**: **End of workout** - Alice evaluates the entire workout session and provides her perceived intensity
**Current Implementation**:

- `calculateAliceIntensity()` in `intensityScoring.ts` just returns `undefined` (stub function)
- Intensity formula: `(tempo × 0.30) + (motionSmoothness × 0.25) + (repConsistency × 0.20) + (userFeedback × 0.15) + (strainModifier × 0.10)`
- `aliceOverall` field exists but is never populated

**What's Missing**:

- Alice should analyze workout context **at end of workout** and provide her perceived intensity (0-100)
- This should be blended with mechanical intensity: `blended = mechanical × (1 - α) + aliceOverall × α`
- Training data needed: Examples showing Alice providing perceived intensity scores **triggered at end of workout** based on overall session context

### 2. Music-Performance Correlation Analysis

**Status**: ❌ **NOT IMPLEMENTED**
**What it should be**: Alice should analyze patterns between music played and performance set-over-set, then create playlists based on performance correlation

**Current Implementation**:

- `StrainPlaylistService` only scores tracks based on user preferences (thumbs up/down, play count)
- No correlation analysis between music and performance
- No playlist creation based on performance patterns

**What's Missing**:

- Alice should analyze: "When Track X plays, user performs Y% better"
- Alice should identify tracks that correlate with:
  - Higher reps at same weight
  - Lower RPE for same effort
  - Better form/consistency
  - Faster recovery
- Alice should create playlists prioritizing tracks with positive performance correlations
- Training data needed: Examples showing Alice analyzing music-performance patterns and creating performance-optimized playlists

**Data Available** (but not being used):

- Set-level music logs: `StrainSyncSetMusicLog` with track info + performance metrics per set
- This data exists in SharedPreferences but is NOT analyzed for correlations

---

## Summary

| Action                          | Status | Location                                                            | Count       | Notes                                                      |
| ------------------------------- | ------ | ------------------------------------------------------------------- | ----------- | ---------------------------------------------------------- |
| Dynamic Rest (user-requested)   | ✅     | comprehensive_agentic_examples.jsonl                                | 17 examples |                                                            |
| Dynamic Rest (strain-based)     | ❌     | Missing                                                             | 0           | **Need examples with strain score triggers**               |
| Dynamic Rest (HR-based, system) | ✅     | all_auto_agentic_examples.jsonl                                     | 3 examples  |                                                            |
| Week-over-week progressions     | ⚠️     | incremental_update_examples.jsonl + all_auto_agentic_examples.jsonl | 7 examples  | **Missing 5-6 week cycles, learning examples**             |
| Mesocycle transitions           | ⚠️     | incremental_update_examples.jsonl                                   | 1 example   | **Missing 5-6 week transitions, learning-based selection** |
| Volume adjustments              | ✅     | all_auto_agentic_examples.jsonl                                     | 3 examples  |                                                            |
| Deload (user-expressed)         | ✅     | comprehensive_agentic_examples.jsonl                                | 30 examples |                                                            |
| Deload (system-triggered)       | ✅     | all_auto_agentic_examples.jsonl                                     | 3 examples  |                                                            |
| Exercise replacements           | ✅     | all_auto_agentic_examples.jsonl                                     | 2 examples  |                                                            |
| Progression engine decisions    | ✅     | all_auto_agentic_examples.jsonl                                     | 4 examples  |                                                            |

## Coverage Status ⚠️

Most automatic agentic actions are covered, but **missing key examples**:

✅ **Covered**:

1. ✅ **System-triggered rest adjustments** (HR-based, automatic) - 3 examples
2. ✅ **Volume adjustment actions** (post-session, guardrail-capped) - 3 examples
3. ✅ **System-triggered deloads** (readiness score, mesocycle complete) - 3 examples
4. ✅ **Exercise replacement actions** (low completion over 4 weeks) - 2 examples
5. ✅ **Progression engine decisions** (completion/effort/recovery-based) - 4 examples
6. ✅ **Week-over-week variations** (different completion rates, exercises) - 3 additional examples

❌ **Missing**:

1. ❌ **Strain score-based rest adjustments** (PRIMARY method) - Need examples with strain triggers (not HR, since HR is already in strain)
2. ❌ **5-6 week mesocycle progressions** - Current examples only cover 4-week cycles
3. ❌ **Learning-based progression selection** - Examples showing Alice choosing which progression type works best
4. ❌ **Single-adjustment constraint examples** - Examples showing only one progression type per week
5. ❌ **Coverage requirement examples** - Examples showing all progression types being tried over time
6. ❌ **Alice's perceived intensity score** - Examples showing Alice providing her perceived intensity **triggered at end of workout** (with trigger context: "Workout session completed. System requesting Alice's perceived intensity evaluation")
7. ❌ **Music-performance correlation analysis** - Examples showing Alice analyzing set-over-set music-performance patterns (requires 30+ sets of data, with trigger: "30+ sets observed, onboarding state active. System requesting music-performance correlation analysis")
8. ❌ **Performance-based playlist creation** - Examples showing Alice creating playlists that balance performance correlation with user musical preferences (requires 30+ sets, respects user preferences first, with trigger: "30+ sets observed, correlation patterns identified. System requesting performance-optimized playlist creation")

## Training Files

All automatic agentic actions are covered in:

- `incremental_update_examples.jsonl` - Week-over-week and mesocycle progressions (4 examples)
- `all_auto_agentic_examples.jsonl` - All system-triggered automatic actions (18 examples)
  - Rest adjustments (HR-based): 3 examples
  - Volume adjustments: 3 examples
  - System deloads: 3 examples
  - Exercise replacements: 2 examples
  - Progression engine: 4 examples
  - Week progression variations: 3 examples

**Total automatic action examples: 22**

## ⚠️ IMPORTANT: Triggers Must Be in Dataset

**Yes, triggers absolutely need to be in the training dataset!**

Alice needs to learn to **recognize trigger conditions** and decide when to take automatic actions. The training examples should include:

1. **Trigger context in SYSTEM CONTEXT field**: The conditions that lead to the action
2. **Variations of trigger conditions**: Different ways the same trigger can appear
3. **Edge cases**: Boundary conditions (e.g., exactly 85% strain, exactly 50% completion)

**Current Examples Include Triggers**:

- ✅ `incremental_update_examples.jsonl` includes: "SYSTEM CONTEXT: User completed week 1 of mesocycle with >85% completion rate"
- ✅ `all_auto_agentic_examples.jsonl` includes: "SYSTEM CONTEXT: Heart rate 92% of max detected during rest period"

**What's Missing**:

- ❌ **More trigger variations**: Different phrasings of the same trigger condition
- ❌ **Boundary conditions**: Exactly at thresholds (85% strain, 50% completion, etc.)
- ❌ **Combined triggers**: Multiple conditions that together trigger an action
- ❌ **Trigger negation examples**: When conditions are NOT met, Alice should NOT take action

**Example of Good Trigger Context**:

```
SYSTEM CONTEXT: Strain score 87% detected during rest period (above 85% threshold). System requesting automatic rest extension.
```

**Example of Missing Trigger Variation**:

```
SYSTEM CONTEXT: Current strain 87, baseline strain 45, recovery delay 35s. Strain calculation: (0.4 × 42) + (0.3 × 3) + (0.3 × 117) = 87%. Above 85% threshold.
```

Both should be in the dataset so Alice learns to recognize the trigger in different contexts.

---

## Next Steps

**Need to generate additional training examples for**:

1. **Strain score-based rest adjustments** (PRIMARY method) - **WITH TRIGGERS**:
   - High strain (>95%) → Extend rest (trigger: "Strain score 97% detected, above 95% threshold")
   - Moderate strain (86-95%) → Slightly longer rest (trigger: "Strain score 88% detected, above 85% threshold")
   - At threshold (85%) → Standard rest (trigger: "Strain score 85% detected, at threshold")
   - Below threshold (<85%) → Standard rest (trigger: "Strain score 78% detected, below 85% threshold")
   - **Include trigger variations**: Different ways to express the same strain condition

2. **Extended mesocycle progressions** (5-6 weeks):
   - Week 4 → Week 5 progressions (with trigger: "User completed week 4 of 6-week mesocycle")
   - Week 5 → Week 6 progressions (with trigger: "User completed week 5 of 6-week mesocycle")
   - 5-6 week mesocycle transitions (with trigger: "User completed 6-week mesocycle")
   - Deload triggers for 6-week mesocycles (with trigger: "Mesocycle complete (week 6/6)")
   - Exercise replacement triggers for 6-week mesocycles (with trigger: "Exercise completion 48% over 6 weeks")

3. **Learning-based progression selection**:
   - Examples showing Alice choosing progression type based on what worked best
   - Examples showing single-adjustment constraint (only one type per week)
   - Examples showing all progression types being tried over multiple mesocycles

**Current training data** (add to enf_train.jsonl):

```bash
cd training/enf_lora
cat data/incremental_update_examples.jsonl >> data/enf_train.jsonl
cat data/all_auto_agentic_examples.jsonl >> data/enf_train.jsonl
```

## Related

^[{src_rel}]
