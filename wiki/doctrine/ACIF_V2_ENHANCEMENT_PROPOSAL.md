---
title: ACIF_V2_ENHANCEMENT_PROPOSAL
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/ACIF_V2_ENHANCEMENT_PROPOSAL.md"]
updated: 2026-07-24
---

# ACIF v2 Enhancement Proposal

## Phase Tracking, Intensity Integration, Nutrition & Supplements

**Date**: 2025-11-05
**Status**: Proposal
**Related Docs**:

- `CONCENTRIC_ECCENTRIC_ANALYSIS.md` (gap analysis)
- `specs/030-acif-federated-learning/data-model.md` (current ACIF v2)
- `specs/003-intensity-score-live/data-model.md` (intensity spec)

---

## 🎯 Executive Summary

This proposal addresses critical gaps in phase tracking and reimagines the ACIF v2 schema to be more efficient and comprehensive by:

1. **Consolidating health data**: Use intensity score (which already aggregates HR, SpO2, recovery) instead of duplicating raw health metrics
2. **Adding phase timing**: Include concentric/eccentric/TUT data for AI learning
3. **Music-intensity correlation**: Keep music data to analyze its effect on intensity
4. **Nutrition integration**: Add macro/calorie data to correlate with performance
5. **Supplement tracking**: Include stack changes to detect performance impacts

---

## 💡 Key Insight: Intensity as Consolidated Health Metric

### Current Problem (Duplication):

```typescript
// ❌ OLD APPROACH: Duplicating health data
interface ACIFSignalPackage {
  signals: {
    hr_delta: number; // Raw health data
    spo2_delta: number; // Raw health data
    // ... plus intensity calculation uses same data
  };
}
```

### Proposed Solution (Consolidation):

```typescript
// ✅ NEW APPROACH: Intensity consolidates health data
interface ACIFSignalPackage {
  signals: {
    intensity_score: number; // 0-100, already includes HR, SpO2, recovery
    intensity_breakdown: {
      // Component scores for granular learning
      tempo: number;
      smoothness: number;
      consistency: number;
      strain_modifier: number;
    };
    // ... other unique signals
  };
}
```

**Benefits**:

- ✅ No data duplication
- ✅ Privacy-preserving (intensity is more abstract than raw HR/SpO2)
- ✅ Simpler signal package (fewer fields)
- ✅ AI learns from processed metric, not raw sensor data

---

## 📋 Enhanced ACIF v2 Schema Proposal

### Complete Updated Schema

```typescript
interface ACIFSignalPackage {
  // Metadata
  app: "fit" | "mind" | "echo";
  mode: "workout" | "journal" | "behavior";
  timestamp: string; // ISO 8601

  signals: {
    // === INTENSITY & PERFORMANCE === //
    intensity_score: number; // 0-100 (consolidated health metric)
    intensity_breakdown: {
      tempo: number; // 0-100
      smoothness: number; // 0-100
      consistency: number; // 0-100
      strain_modifier: number; // 0.85, 0.95, or 1.0
    };

    // === PHASE TIMING (NEW) === //
    phase_timing?: {
      // Only present during resistance training
      concentric_avg_ms: number; // Average concentric time across reps
      eccentric_avg_ms: number; // Average eccentric time across reps
      pause_avg_ms: number; // Average pause time between phases
      total_tut_ms: number; // Total time under tension for set
      tempo_variance: number; // Deviation from target (0-100, lower is better)
      is_eccentric_first: boolean; // Exercise type flag
    };

    // === MUSIC CORRELATION === //
    music_bpm?: number; // Music tempo during workout
    music_genre?: string; // Anonymized genre category
    music_energy?: number; // 0-100 energy level from music analysis

    // === NUTRITION (NEW) === //
    nutrition?: {
      pre_workout_calories?: number; // Calories consumed 2h before
      pre_workout_carbs?: number; // Grams of carbs
      pre_workout_protein?: number; // Grams of protein
      hydration_ml?: number; // Water intake in ml
      caffeine_mg?: number; // Caffeine consumption
      time_since_meal_mins?: number; // Minutes since last meal
    };

    // === SUPPLEMENT TRACKING (NEW) === //
    supplement_stack?: {
      stack_hash: string; // Hashed identifier for supplement combination
      days_on_stack: number; // Days since stack started/changed
      stack_changed: boolean; // True if stack was modified today
      categories: string[]; // Anonymized categories (e.g., "pre-workout", "creatine", "bcaa")
      // NOTE: No specific product names or Rx compounds
    };

    // === CONTEXT & MOOD === //
    mood_predicted: number; // 1-5 scale (from physiological signals)
    fatigue?: number; // 1-5 scale (user reported or AI predicted)
    location_category: string; // User-defined (e.g., "gym", "friend_1")

    // === EXERCISE CONTEXT === //
    reps_delta?: number; // Change in reps from previous workout
    volume_delta?: number; // Change in volume (weight × reps)
    rest_time_ms?: number; // Rest period duration
  };
}
```

---

## 🔄 Data Flow: From Sensors to ACIF

### 1. Workout Data Collection

```
Motion Sensors (Accelerometer, Gyro)
  ↓
Phase Detection Service (NEW)
  ↓
Phase Timing Data {concentric, eccentric, pause, TUT}
  ↓
Intensity Scoring Service (UPDATED)
  ↓
Intensity Score + Breakdown {tempo, smoothness, consistency, strain}
  ↓
ACIF Signal Package
```

### 2. Health Data Flow (Consolidated)

```
Wearable (HR, SpO2, HRV)
  ↓
Strain Calculator
  ↓
Strain Modifier (0.85, 0.95, 1.0)
  ↓
Intensity Score (via intensity_breakdown.strain_modifier)
  ↓
ACIF Signal Package (NO raw HR/SpO2)
```

### 3. Nutrition & Supplement Flow (NEW)

```
Nutrition Tracking (Pre-Workout Meal)
  ↓
Nutrition Preprocessor
  ↓
ACIF nutrition object {calories, macros, hydration, timing}

Supplement Stack Scanner
  ↓
Stack Hash Generator (privacy-preserving)
  ↓
ACIF supplement_stack object {stack_hash, days_on_stack, categories}
```

---

## 🔄 Graceful Degradation & Fallback Strategies

### Overview

The intensity calculation must work reliably even when some data inputs are missing. This section defines the fallback hierarchy and degradation strategies.

### Intensity Calculation Formula (Full Data)

```typescript
intensity_score =
  (tempo × 0.30) +           // Phase timing + HR consistency
  (smoothness × 0.25) +      // Motion quality
  (consistency × 0.20) +     // Rep-to-rep consistency
  (userFeedback × 0.15) +    // User perceived effort
  (strainModifier × 0.10)    // HR rise + SpO2 drop + recovery
```

### Data Availability Scenarios

| Scenario                       | Motion Sensors | Wearable (HR/SpO2) | User Feedback | Strategy                   |
| ------------------------------ | -------------- | ------------------ | ------------- | -------------------------- |
| 1. **Full Data**               | ✅             | ✅                 | ✅            | Full intensity calculation |
| 2. **No Motion**               | ❌             | ✅                 | ✅            | HR-based tempo estimation  |
| 3. **No Wearable**             | ✅             | ❌                 | ✅            | Motion-only calculation    |
| 4. **No Feedback**             | ✅             | ✅                 | ❌            | Default to neutral (50%)   |
| 5. **Motion + Feedback Only**  | ✅             | ❌                 | ✅            | Estimated intensity        |
| 6. **Wearable Only**           | ❌             | ✅                 | ❌            | HR zones only              |
| 7. **Minimal (Feedback Only)** | ❌             | ❌                 | ✅            | User perception only       |

---

### Scenario 1: Full Data Available ✅

**Data Available**:

- Motion sensors (accelerometer, gyro)
- Wearable (HR, SpO2)
- User feedback

**Calculation**:

```typescript
intensity_score =
  (phase_aware_tempo × 0.30) +
  (motion_smoothness × 0.25) +
  (rep_consistency × 0.20) +
  (user_feedback × 0.15) +
  (strain_modifier × 0.10)
```

**ACIF Signals**:

- Full `intensity_breakdown` with all components
- `phase_timing` object present
- `is_estimated: false`

---

### Scenario 2: No Motion Sensors ❌✅✅

**Data Available**:

- Wearable (HR, SpO2) ✅
- User feedback ✅
- Motion sensors ❌

**Why This Happens**:

- User's phone is in pocket/locker during workout
- Wearable without phone nearby
- Motion API unavailable (permissions denied)

**Fallback Strategy**:

1. **Tempo Score**: Use HR variability as proxy

   ```typescript
   // Fall back to HR-based tempo
   tempo_score = calculateTempoFromHR(heartRateData);
   // HR consistency over time replaces phase timing
   ```

2. **Smoothness Score**: Estimate from HR pattern

   ```typescript
   // Smooth HR increases = controlled movement
   smoothness_score = calculateHRSmoothness(heartRateData);
   // Jerky HR spikes = rough movement
   ```

3. **Consistency Score**: Estimate from HR pattern

   ```typescript
   // Consistent HR plateaus between sets = consistent reps
   consistency_score = calculateHRConsistency(heartRateData);
   ```

4. **Weights Rebalanced**:
   ```typescript
   intensity_score =
     (tempo × 0.25) +          // Reduced weight (HR proxy)
     (smoothness × 0.20) +     // Reduced weight (estimated)
     (consistency × 0.15) +    // Reduced weight (estimated)
     (userFeedback × 0.25) +   // INCREASED weight (more reliable)
     (strainModifier × 0.15)   // INCREASED weight (wearable data reliable)
   ```

**ACIF Signals**:

- `phase_timing`: null (not available)
- `intensity_breakdown.tempo`: estimated from HR
- `is_estimated: true`
- Label: "Estimated (No motion data)"

---

### Scenario 3: No Wearable ❌✅❌

**Data Available**:

- Motion sensors ✅
- User feedback ✅
- Wearable ❌

**Why This Happens**:

- User doesn't own a wearable
- Wearable battery died
- Bluetooth connection lost
- User preference (privacy)

**Fallback Strategy**:

1. **Tempo Score**: Full phase timing analysis

   ```typescript
   // Full phase detection from motion sensors
   tempo_score = calculatePhaseAwareTempo(motionData);
   // No change, this is preferred method
   ```

2. **Smoothness Score**: Full motion analysis

   ```typescript
   // Calculate from accelerometer jerk
   smoothness_score = calculateMotionSmoothness(motionData);
   // No change, this is preferred method
   ```

3. **Consistency Score**: Full motion analysis

   ```typescript
   // Rep detection from motion patterns
   consistency_score = calculateRepConsistency(motionData);
   // No change, this is preferred method
   ```

4. **Strain Modifier**: Estimate from motion + feedback

   ```typescript
   // Use acceleration magnitude as proxy for exertion
   estimated_strain = calculateMotionBasedStrain(motionData);

   // Cross-reference with user feedback
   if (userFeedback === "easy_killer") {
     estimated_strain = Math.max(estimated_strain, 0.95);
   } else if (userFeedback === "challenge") {
     estimated_strain = Math.min(estimated_strain, 0.85);
   }

   strain_modifier = mapStrainToModifier(estimated_strain);
   ```

5. **Weights Rebalanced**:

   ```typescript
   intensity_score =
     (tempo × 0.35) +          // INCREASED weight (motion is reliable)
     (smoothness × 0.30) +     // INCREASED weight (motion is reliable)
     (consistency × 0.20) +    // Same weight
     (userFeedback × 0.15) +   // Same weight
     (strainModifier × 0.00)   // REMOVED (unreliable without HR/SpO2)

   // Then normalize to 0-100 scale
   intensity_score = (intensity_score / 1.00) * 100;
   ```

**ACIF Signals**:

- `phase_timing`: full data (motion-based)
- `intensity_breakdown.strain_modifier`: 1.0 (default, no strain data)
- `is_estimated: true`
- Label: "Estimated (No wearable)"

---

### Scenario 4: No User Feedback ✅✅❌

**Data Available**:

- Motion sensors ✅
- Wearable ✅
- User feedback ❌

**Why This Happens**:

- User skips post-set feedback
- Automated workout (no prompts)
- Trainer-led session (no time for input)

**Fallback Strategy**:

1. **User Feedback Score**: Default to neutral

   ```typescript
   // Assume neutral perceived effort
   userFeedback_score = 50; // Mid-point of 0-100 scale
   ```

2. **No Weight Rebalancing**:
   ```typescript
   intensity_score =
     (tempo × 0.30) +
     (smoothness × 0.25) +
     (consistency × 0.20) +
     (50 × 0.15) +            // Neutral feedback
     (strainModifier × 0.10)
   ```

**ACIF Signals**:

- All other signals normal
- `intensity_breakdown.userFeedback`: 50 (neutral)
- `is_estimated: false` (objective data is reliable)
- Label: "No user feedback"

---

### Scenario 5: Motion + Feedback Only ✅❌✅

**Data Available**:

- Motion sensors ✅
- User feedback ✅
- Wearable ❌

**Fallback Strategy**:

Same as Scenario 3 (No Wearable), but with user feedback to cross-validate motion-based strain estimation.

```typescript
intensity_score =
  (tempo × 0.40) +          // INCREASED (motion reliable)
  (smoothness × 0.30) +     // INCREASED (motion reliable)
  (consistency × 0.20) +    // Same
  (userFeedback × 0.10)     // DECREASED (less weight needed with good motion data)
```

**ACIF Signals**:

- `phase_timing`: full data
- `is_estimated: true`
- Label: "Motion + user feedback"

---

### Scenario 6: Wearable Only ❌✅❌

**Data Available**:

- Wearable ✅
- Motion sensors ❌
- User feedback ❌

**Why This Happens**:

- Standalone wearable workout (no phone)
- Motion permissions denied
- Minimal UI interaction

**Fallback Strategy**:

1. **HR Zone-Based Intensity**:

   ```typescript
   // Simplified intensity based on HR zones
   const maxHR = calculateMaxHR(userAge);
   const avgHR = calculateAverageHR(heartRateData);
   const hrPercent = (avgHR / maxHR) * 100;

   // Map HR zones to intensity
   if (hrPercent < 60)
     intensity_score = 30; // Very light
   else if (hrPercent < 70)
     intensity_score = 50; // Light
   else if (hrPercent < 80)
     intensity_score = 70; // Moderate
   else if (hrPercent < 90)
     intensity_score = 85; // Hard
   else intensity_score = 95; // Very hard
   ```

2. **Minimal ACIF Signals**:
   ```typescript
   signals: {
     intensity_score: hrZoneIntensity,
     intensity_breakdown: {
       tempo: 0,           // Unknown
       smoothness: 0,      // Unknown
       consistency: 0,     // Unknown
       strain_modifier: calculateStrainModifier(hrData, spo2Data)
     },
     is_estimated: true
   }
   ```

**ACIF Signals**:

- Minimal data (intensity_score only)
- `phase_timing`: null
- `is_estimated: true`
- Label: "HR zones only"

---

### Scenario 7: Minimal (Feedback Only) ❌❌✅

**Data Available**:

- User feedback ✅
- Motion sensors ❌
- Wearable ❌

**Why This Happens**:

- Manual workout logging (post-workout entry)
- No sensors available
- Legacy workout entry

**Fallback Strategy**:

1. **User Perception Only**:

   ```typescript
   // Map user feedback to intensity
   const feedbackMap = {
     easy_killer: 30, // Too easy
     neutral: 50, // Just right
     finally_challenge: 70, // Good challenge
     flag_review: 40, // Something wrong
   };

   intensity_score = feedbackMap[userFeedback] || 50;
   ```

2. **Minimal ACIF Signals**:
   ```typescript
   signals: {
     intensity_score: feedbackScore,
     intensity_breakdown: null, // No breakdown available
     is_estimated: true
   }
   ```

**ACIF Signals**:

- Only `intensity_score` (no breakdown)
- `phase_timing`: null
- `is_estimated: true`
- Label: "User feedback only"

---

## 📊 Fallback Quality Indicators

### Confidence Scores

Each intensity calculation includes a confidence score based on available data:

```typescript
interface IntensityResult {
  score: number; // 0-100
  confidence: number; // 0-100, how confident we are
  data_sources: string[]; // ['motion', 'wearable', 'feedback']
  is_estimated: boolean;
  estimation_method: string; // 'full', 'hr_proxy', 'motion_only', etc.
  missing_inputs: string[]; // ['phase_timing', 'hr_data', etc.]
}
```

### Confidence Calculation

```typescript
function calculateConfidence(availableInputs: string[]): number {
  let confidence = 0;

  // Motion sensors (phase timing): +40%
  if (availableInputs.includes("motion")) confidence += 40;

  // Wearable (HR, SpO2): +35%
  if (availableInputs.includes("wearable")) confidence += 35;

  // User feedback: +15%
  if (availableInputs.includes("feedback")) confidence += 15;

  // Bonus for complete data: +10%
  if (confidence === 90) confidence += 10; // 100% with all three

  return confidence;
}
```

**Confidence Thresholds**:

- **100%**: All inputs available (motion + wearable + feedback)
- **75-90%**: Two of three inputs available
- **50-74%**: One primary input (motion or wearable)
- **<50%**: Feedback only or legacy data

---

## 🎯 UI Indicators for Missing Data

### User Experience

When data is missing, show clear indicators:

1. **Intensity Display Badge**:

   ```
   Intensity: 78%
   [Estimated - No wearable] ⓘ
   ```

2. **Tooltip Explanation**:

   ```
   "This intensity score is estimated from motion sensors and
   your feedback. Connect a wearable for more accurate scores."
   ```

3. **Upgrade Prompts** (non-intrusive):

   ```
   "💡 Get more accurate intensity scores with a compatible wearable"
   [Learn More] [Dismiss]
   ```

4. **Trainer Dashboard Indicators**:
   ```
   Client Intensity: 82% ⚠️ (Motion only)
   ```

---

## 🔧 Implementation Guidelines

### Service Layer

```typescript
// app/src/lib/services/intensityScoring.ts

export class IntensityScoring {
  calculateIntensityScore(inputs: IntensityInputs): IntensityResult {
    // Detect available data
    const hasMotion = inputs.motionData && inputs.motionData.length > 0;
    const hasWearable = inputs.heartRateData && inputs.heartRateData.length > 0;
    const hasFeedback = inputs.userFeedback !== undefined;

    // Route to appropriate calculation method
    if (hasMotion && hasWearable && hasFeedback) {
      return this.calculateFullIntensity(inputs);
    } else if (!hasMotion && hasWearable && hasFeedback) {
      return this.calculateHRBasedIntensity(inputs);
    } else if (hasMotion && !hasWearable) {
      return this.calculateMotionOnlyIntensity(inputs);
    } else if (!hasMotion && !hasWearable && hasFeedback) {
      return this.calculateFeedbackOnlyIntensity(inputs);
    } else {
      return this.calculateMinimalIntensity(inputs);
    }
  }

  private calculateFullIntensity(inputs: IntensityInputs): IntensityResult {
    // Full calculation with all data
    const breakdown = {
      tempo: this.calculatePhaseAwareTempo(
        inputs.motionData,
        inputs.phaseTiming,
      ),
      smoothness: this.calculateMotionSmoothness(inputs.motionData),
      consistency: this.calculateRepConsistency(inputs.motionData),
      userFeedback: this.mapUserFeedback(inputs.userFeedback),
      strainModifier: this.calculateStrainModifier(
        inputs.heartRateData,
        inputs.spo2Data,
      ),
    };

    const score =
      breakdown.tempo * 0.3 +
      breakdown.smoothness * 0.25 +
      breakdown.consistency * 0.2 +
      breakdown.userFeedback * 0.15 +
      breakdown.strainModifier * 0.1;

    return {
      score: Math.round(score),
      confidence: 100,
      data_sources: ["motion", "wearable", "feedback"],
      is_estimated: false,
      estimation_method: "full",
      missing_inputs: [],
      breakdown,
    };
  }

  // ... other calculation methods
}
```

---

## 📋 ACIF Schema Updates for Fallback

Add metadata to ACIF signals to indicate data quality:

```typescript
interface ACIFSignalPackage {
  // ... existing fields

  metadata: {
    intensity_confidence: number; // 0-100
    data_sources: string[]; // ['motion', 'wearable', 'feedback']
    is_estimated: boolean;
    missing_inputs: string[]; // ['phase_timing', 'hr_data']
    estimation_method: string; // 'full', 'hr_proxy', 'motion_only'
  };
}
```

**Federated Learning Use**:

- Weight higher-confidence signals more heavily during aggregation
- Filter out <50% confidence signals for critical patterns
- Use estimated signals for broad trend analysis only

---

## ✅ Testing Strategy for Fallbacks

### Unit Tests

```typescript
describe("IntensityScoring Fallbacks", () => {
  it("should calculate full intensity with all data", () => {
    const result = intensityScoring.calculateIntensityScore({
      motionData: mockMotionData,
      heartRateData: mockHRData,
      userFeedback: "neutral",
    });

    expect(result.confidence).toBe(100);
    expect(result.is_estimated).toBe(false);
  });

  it("should fall back to HR-based when motion missing", () => {
    const result = intensityScoring.calculateIntensityScore({
      motionData: null,
      heartRateData: mockHRData,
      userFeedback: "neutral",
    });

    expect(result.confidence).toBeLessThan(100);
    expect(result.is_estimated).toBe(true);
    expect(result.estimation_method).toBe("hr_proxy");
  });

  it("should fall back to motion-only when wearable missing", () => {
    const result = intensityScoring.calculateIntensityScore({
      motionData: mockMotionData,
      heartRateData: null,
      userFeedback: "neutral",
    });

    expect(result.breakdown.strainModifier).toBe(1.0); // Default
    expect(result.is_estimated).toBe(true);
  });
});
```

---

## 🛠️ Implementation Plan

### Phase 1: Core Phase Detection (Week 1-2)

**Priority**: Critical

**Tasks**:

1. Create `phaseDetectionService.ts`
   - Input: Motion sensor data (accelerometer, gyroscope)
   - Output: Phase transitions (pause → concentric → pause → eccentric)
   - Algorithm: Peak detection + direction analysis
2. Calculate split times
   - Track duration of each phase per rep
   - Calculate averages across set
   - Compute total TUT
3. Implement eccentric-first detection
   - Toggle in exercise settings
   - Reverse phase detection logic when enabled
4. Unit tests
   - Mock motion data for various exercises
   - Test phase transition accuracy
   - Edge cases (incomplete reps, pauses mid-rep)

**Files to Create**:

- `app/src/lib/services/phaseDetectionService.ts`
- `app/src/lib/services/__tests__/phaseDetectionService.test.ts`

**Files to Update**:

- `app/src/lib/services/intensityScoring.ts` (integrate phase timing)

---

### Phase 2: Intensity Integration (Week 2-3)

**Priority**: Critical

**Tasks**:

1. Update intensity scoring to use phase timing
   - Replace HR-based tempo score with phase-aware tempo score
   - Compare actual vs target tempo patterns
   - Calculate variance (±15% tolerance)
2. Update ACIF signal generation
   - Replace `hr_delta` and `spo2_delta` with `intensity_score`
   - Add `intensity_breakdown` object
   - Add `phase_timing` object
3. Full strain formula implementation
   - `(0.4 × HR rise) + (0.3 × SpO₂ drop) + (0.3 × recovery delay)`
   - Update `strainCalculator.ts`
4. Integration tests
   - End-to-end: sensors → phase detection → intensity → ACIF
   - Verify no raw health data in ACIF package

**Files to Update**:

- `app/src/lib/services/intensityScoring.ts`
- `app/src/lib/services/strainCalculator.ts`
- `specs/030-acif-federated-learning/data-model.md`

---

### Phase 3: UI Components (Week 3-4)

**Priority**: High

**Tasks**:

1. Create phase timing display component
   - Show concentric/eccentric/pause split times
   - Visual TUT indicator
   - Tempo variance feedback
2. Update intensity display
   - Add phase timing section
   - Show target vs actual tempo
   - Color-coded variance indicator
3. Trainer dashboard enhancements
   - Live split time display (FR-018 requirement)
   - Historical phase timing trends
   - Client tempo consistency charts
4. Exercise settings UI
   - Target tempo input ("3-1-2-1" format)
   - Eccentric-first toggle
   - Tempo mode enable/disable

**Files to Create**:

- `app/src/lib/components/intensity/PhaseTimingDisplay.svelte`
- `app/src/lib/components/intensity/TempoTargetInput.svelte`

**Files to Update**:

- `app/src/lib/components/intensity/IntensityDisplay.svelte`
- `app/src/lib/components/intensity/IntensityConfig.svelte`
- `app/src/lib/components/trainer/RealtimeWorkoutView.svelte`

---

### Phase 4: Nutrition Integration (Week 4-5)

**Priority**: Medium

**Tasks**:

1. Create nutrition preprocessor
   - Extract pre-workout meal data (2h window)
   - Calculate macros (carbs, protein, fats)
   - Track hydration and caffeine
   - Calculate time since last meal
2. Add nutrition to ACIF signals
   - Privacy check (no specific foods, only macros)
   - Anonymize meal timing
   - Include in signal package
3. Nutrition-intensity correlation analysis
   - Track intensity performance vs nutrition patterns
   - Identify optimal pre-workout nutrition
   - Generate personalized recommendations
4. UI for nutrition tracking (if not already existing)
   - Quick log pre-workout meal
   - Macro calculator
   - Hydration tracker

**Files to Create**:

- `app/src/lib/services/nutritionPreprocessor.ts`
- `app/src/lib/services/nutritionIntensityAnalyzer.ts`

**Files to Update**:

- `specs/030-acif-federated-learning/data-model.md` (add nutrition fields)

---

### Phase 5: Supplement Stack Tracking (Week 5-6)

**Priority**: Medium

**Tasks**:

1. Create stack hash generator
   - Privacy-preserving hash of supplement combination
   - Exclude Rx compounds from hash
   - Track stack change events
2. Add supplement stack to ACIF signals
   - Include stack_hash, days_on_stack, categories
   - NO specific product names or dosages
   - Flag when stack changes
3. Supplement-intensity correlation analysis
   - Track intensity trends before/after stack changes
   - 28-day comparison window (matches spec 003 lock period)
   - Detect positive/negative performance shifts
4. UI integration
   - Show stack change impact on intensity
   - Alert if performance degrades >8% after stack change
   - Visualize intensity trends during stack period

**Files to Create**:

- `app/src/lib/services/supplementStackHasher.ts`
- `app/src/lib/services/supplementIntensityAnalyzer.ts`

**Files to Update**:

- `specs/030-acif-federated-learning/data-model.md` (add supplement fields)

---

### Phase 6: Federated Learning Updates (Week 6-7)

**Priority**: High (for ACIF v2 effectiveness)

**Tasks**:

1. Update Fly.io aggregation service
   - Accept new ACIF schema with phase_timing, nutrition, supplement_stack
   - Validate signal packages against updated schema
   - Aggregate across updated fields
2. Update Alice/Aiden AI models
   - Train on phase timing patterns
   - Learn nutrition-intensity correlations
   - Detect supplement impact patterns
   - Music-intensity relationships
3. Model update testing
   - Canary testing with new ACIF signals
   - Validate federated aggregation accuracy
   - Rollback procedures if needed
4. Privacy audit
   - Verify no PII in new fields
   - Confirm stack_hash cannot be reverse-engineered
   - Check nutrition data anonymization

**Files to Update**:

- Fly.io aggregation service (outside this repo)
- `specs/030-acif-federated-learning/data-model.md` (finalize schema)

---

## 📊 Updated ACIF Privacy Guarantees

### What's Included ✅

- **Intensity score**: Aggregated performance metric (0-100)
- **Phase timing**: Average times (no specific timestamps)
- **Music**: BPM, genre, energy (no song titles/artists)
- **Nutrition**: Macros and timing (no specific foods)
- **Supplements**: Hashed stack + categories (no product names, no Rx)
- **Context**: Location category (no GPS), mood prediction (no raw data)

### What's Excluded ❌

- **Raw health data**: No HR, SpO2, HRV (use intensity instead)
- **Identifiers**: No device IDs, user names, GPS coordinates
- **Personal content**: No workout notes, journal entries, chat messages
- **Medical info**: No Rx compounds, no specific medications
- **Brand data**: No supplement product names, no food brands

### Privacy Improvements vs Old ACIF

| Old ACIF         | New ACIF                                    | Privacy Benefit                           |
| ---------------- | ------------------------------------------- | ----------------------------------------- |
| `hr_delta: 25`   | `intensity_score: 78`                       | More abstract, harder to reverse-engineer |
| `spo2_delta: -3` | `intensity_breakdown.strain_modifier: 0.95` | No raw physiological data                 |
| N/A              | `nutrition.pre_workout_carbs: 45`           | Macros only, no food names                |
| N/A              | `supplement_stack.stack_hash: "a3f9..."`    | Cannot identify specific products         |

---

## 🎯 Success Metrics

### Technical Metrics

1. **Phase Detection Accuracy**: >90% correct phase identification
2. **Tempo Variance Calculation**: Within ±5% of manual measurement
3. **ACIF Package Size**: <3KB (was <2KB, adding new fields)
4. **Privacy Compliance**: Zero PII leakage in automated testing
5. **Model Improvement**: >10% better intensity predictions after 4 weeks

### User Experience Metrics

1. **Trainer Adoption**: 70% of trainers use live split time display
2. **User Engagement**: 40% of users configure target tempo patterns
3. **Nutrition Correlation**: Users with tracked nutrition see 15% better intensity consistency
4. **Supplement Impact Detection**: 80% of stack changes detected within 7 days

### Business Metrics

1. **AI Accuracy**: Mood prediction accuracy improves to 80% (from 75% baseline)
2. **Retention**: Multi-signal users (nutrition + supplements + phase) retain 2x longer
3. **Premium Conversion**: Phase timing features drive 20% conversion lift
4. **Trainer Revenue**: Data flywheel participation increases 30%

---

## 🔒 Security & Privacy Considerations

### 1. Stack Hash Security

**Threat**: Could someone reverse-engineer a stack_hash to identify products?
**Mitigation**:

- Use salted SHA-256 with user-specific salt
- Hash only anonymized categories, not product names
- Exclude Rx compounds entirely from hash
- Rotate salts quarterly

### 2. Nutrition Data Privacy

**Threat**: Could macro patterns reveal eating disorders or medical conditions?
**Mitigation**:

- Only include pre-workout window (2h before)
- Aggregate to daily totals (not meal-by-meal)
- Cap extreme values (no <500 or >5000 cal/day signals)
- User can disable nutrition signals entirely

### 3. Phase Timing Fingerprinting

**Threat**: Could unique phase timing patterns identify individuals?
**Mitigation**:

- Round all times to nearest 100ms
- Average across sets (not individual reps)
- Exclude outlier reps (>2 std dev from mean)
- Federated aggregation across 100+ users minimum

### 4. Intensity Score Reconstruction

**Threat**: Could intensity breakdown be used to reconstruct raw HR/SpO2?
**Mitigation**:

- Breakdown only shows component scores (0-100), not raw data
- Strain modifier is categorical (0.85, 0.95, 1.0), not continuous
- No timestamps for individual measurements
- Aggregated over entire set, not rep-by-rep

---

## 🧪 Testing Strategy

### Unit Tests

- Phase detection algorithm (various exercise types)
- Stack hash generation (uniqueness, collision resistance)
- Nutrition preprocessor (macro calculation accuracy)
- Tempo variance calculation (±15% tolerance)

### Integration Tests

- End-to-end: sensors → ACIF signal package
- Privacy validation: scan for PII in ACIF packages
- Schema validation: all fields conform to spec
- Data flow: nutrition → intensity correlation

### User Testing (Alpha)

- 20 trainers test live split time display
- 50 users test target tempo configuration
- 30 users test nutrition-intensity tracking
- Collect feedback on UI/UX

### Performance Testing

- Phase detection latency: <100ms
- ACIF package generation: <50ms
- Federated aggregation: handles 10,000 packages/week

---

## 📅 Timeline Summary

| Phase                    | Duration  | Priority | Deliverables                     |
| ------------------------ | --------- | -------- | -------------------------------- |
| 1. Phase Detection       | 2 weeks   | Critical | Service, tests, integration      |
| 2. Intensity Integration | 2 weeks   | Critical | Updated scoring, ACIF schema     |
| 3. UI Components         | 2 weeks   | High     | Phase display, trainer dashboard |
| 4. Nutrition Integration | 1.5 weeks | Medium   | Preprocessor, ACIF nutrition     |
| 5. Supplement Tracking   | 1.5 weeks | Medium   | Stack hasher, ACIF supplements   |
| 6. Federated Learning    | 1 week    | High     | Aggregation, model updates       |

**Total**: ~10 weeks (2.5 months)

---

## 🤔 Open Questions

1. **Phase Detection Hardware**:
   - Which wearables support accelerometer/gyroscope access?
   - What's the sampling rate? (Need ≥50Hz for accurate phase detection)
   - iOS vs Android sensor API differences?

2. **Nutrition Data Source**:
   - Should we integrate with MyFitnessPal/Cronometer APIs?
   - Or build native nutrition logging?
   - Pre-workout meal only, or full day tracking?

3. **Supplement Stack Changes**:
   - How do we handle gradual dosage changes vs complete stack swaps?
   - Should "stack_changed" flag trigger only for major changes?
   - What constitutes a "major change"? (50% of stack? New category added?)

4. **Music Integration**:
   - Do we have music API access for BPM/energy analysis?
   - Or rely on user-reported music genre?
   - Should we integrate with Spotify/Apple Music APIs?

5. **Federated Learning Timeline**:
   - How often should ACIF signals be uploaded? (Daily? Weekly?)
   - What's the minimum sample size for meaningful aggregation?
   - Canary testing duration for new model versions?

---

## ✅ Approval Checklist

- [ ] Technical review: Architecture approved
- [ ] Privacy review: No PII concerns
- [ ] Security review: Stack hashing secure
- [ ] UX review: Trainer/user UI designs approved
- [ ] Legal review: GDPR/CCPA compliance verified
- [ ] Business review: ROI projections acceptable
- [ ] Spec update: ACIF v2 schema finalized in spec 030
- [ ] Timeline approval: 10-week timeline feasible

---

## 📝 Next Steps

1. **Review this proposal** with product, engineering, and privacy teams
2. **Prioritize phases** based on user impact and technical dependencies
3. **Create Jira tickets** for each phase
4. **Update specs** (030-acif-federated-learning, 003-intensity-score-live)
5. **Design UI mockups** for phase timing display and trainer dashboard
6. **Research sensors** - confirm accelerometer/gyroscope access on target devices
7. **Prototype phase detection** with mock data to validate algorithm

---

**Document Owner**: AI Assistant
**Last Updated**: 2025-11-05
**Status**: Awaiting Review
**Stakeholders**: Product, Engineering, Privacy, Legal, UX

## Related

^[{src_rel}]
