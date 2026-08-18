---
title: CONCENTRIC_ECCENTRIC_ANALYSIS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CONCENTRIC_ECCENTRIC_ANALYSIS.md"]
updated: 2026-07-24
---

# Concentric/Eccentric Phase & Intensity Calculation Analysis

**Date**: 2025-11-05
**Status**: Gap Analysis Complete

## Executive Summary

After reviewing the codebase, ACIF v2 spec (030), and Intensity Score Live spec (003), I've identified **critical gaps** in the implementation of concentric/eccentric phase tracking and intensity calculation.

---

## ✅ What's Specified (Requirements)

### From Spec 003 (Intensity Score Live):

1. **FR-016**: System MUST identify first movement after pause as concentric, reverse as eccentric
2. **FR-017**: System MUST score tempo within ±15% of target values
3. **FR-018**: Trainers MUST see live split times for eccentric/concentric phases
4. **FR-019**: System MUST provide toggle for eccentric-first movement patterns

### From Data Model (003):

- **TempoData** entity with fields:
  - `concentricTimeMs`: number - time for concentric phase
  - `eccentricTimeMs`: number - time for eccentric phase
  - `pauseTimeMs`: number - time spent in paused position
  - `targetTempoPattern`: string - intended tempo (e.g., "3-1-2-1")
  - `actualTempoPattern`: string - measured tempo pattern
  - `tempoVariance`: number - deviation from target (±15% acceptable)
  - `isEccentricFirst`: boolean - whether movement started eccentric

---

## ❌ What's Missing in Implementation

### 1. **Phase Detection Logic** (Not Implemented)

**Current Implementation** (`app/src/lib/services/intensityScoring.ts`):

- `calculateTempoScore()` only uses **heart rate patterns**
- No motion sensor analysis for phase detection
- No identification of "first movement after pause" as concentric
- No detection of reverse movement as eccentric

**What Should Exist**:

```typescript
// Missing: Phase detection from motion data
detectPhaseTransition(motionData: MotionData[]): {
  phase: 'concentric' | 'eccentric' | 'pause';
  timestamp: number;
  duration: number;
}
```

### 2. **Split Time Tracking** (Not Implemented)

**Current Implementation**:

- No `concentricTimeMs` or `eccentricTimeMs` calculation
- No split time display in UI components
- Trainers cannot see live phase timings

**What Should Exist**:

- Real-time split time tracking per rep
- Display in trainer UI showing:
  - Concentric time: `2.3s`
  - Eccentric time: `3.1s`
  - Pause time: `0.5s`
  - Total TUT: `5.9s`

### 3. **Eccentric-First Toggle** (Not in UI)

**Current Implementation**:

- No toggle in UI components
- No `isEccentricFirst` flag handling
- No logic to adjust phase detection for eccentric-first exercises (e.g., Romanian Deadlifts)

**What Should Exist**:

- Toggle in workout settings: "Eccentric-First Movement"
- When enabled, reverses phase detection logic
- Updates tempo calculation accordingly

### 4. **Tempo Variance Calculation** (Incomplete)

**Current Implementation**:

- Tempo score is based on heart rate consistency
- No comparison to target tempo pattern
- No ±15% variance check against target

**What Should Exist**:

```typescript
// Missing: Target tempo comparison
calculateTempoVariance(
  targetPattern: "3-1-2-1",  // e.g., 3s concentric, 1s pause, 2s eccentric, 1s pause
  actualPattern: "3.2-0.9-2.1-1.0"
): {
  variance: number;  // Percentage deviation
  withinTolerance: boolean;  // ±15% check
  score: number;  // 0-100 based on variance
}
```

### 5. **ACIF v2 Signal Package** (Missing Phase Data)

**Current ACIF Signal Package** (`specs/030-acif-federated-learning/data-model.md`):

```typescript
interface ACIFSignalPackage {
  signals: {
    hr_delta: number;
    spo2_delta: number;
    mood_predicted: number;
    location_category: string;
    music_bpm?: number;
    reps_delta?: number; // ✅ Present
    fatigue?: number;
    // ❌ MISSING: Phase timing data
    // concentric_avg_ms?: number;
    // eccentric_avg_ms?: number;
    // tempo_variance?: number;
  };
}
```

**Gap**: Phase timing data is not included in ACIF signals for federated learning, so the AI cannot learn from tempo patterns across users.

---

## 🔍 UI Components Analysis

### Current UI Components (`app/src/lib/components/intensity/`):

1. **IntensityDisplay.svelte** ✅
   - Shows overall intensity score
   - Shows breakdown (tempo, smoothness, consistency, feedback)
   - ❌ **Missing**: Split time display for trainers
   - ❌ **Missing**: Phase timing breakdown

2. **IntensityIntegration.svelte** ✅
   - Integrates intensity scoring with workout flow
   - ❌ **Missing**: Eccentric-first toggle
   - ❌ **Missing**: Phase detection controls

3. **IntensityConfig.svelte** ✅
   - Configures intensity weights
   - ❌ **Missing**: Tempo pattern configuration (target tempo input)
   - ❌ **Missing**: Eccentric-first toggle

4. **Trainer Components** (`app/src/lib/components/trainer/`):
   - **RealtimeWorkoutView.svelte** shows intensity score
   - ❌ **Missing**: Live split time display as specified in FR-018

---

## 📊 Intensity Calculation Issues

### Current Formula (from `intensityScoring.ts`):

```typescript
overall =
  tempo * 0.3 + // ❌ Based on HR, not phase timing
  motionSmoothness * 0.25 + // ✅ Implemented
  repConsistency * 0.2 + // ✅ Implemented
  userFeedback * 0.15 + // ✅ Implemented
  strainModifier * 0.1; // ⚠️ Simplified (not using full strain formula)
```

### Spec Requirement (from 003):

```typescript
intensity =
  (tempo × 0.30) +           // ❌ Should be phase-aware tempo vs target
  (motionSmoothness × 0.25) + // ✅ Correct
  (repConsistency × 0.20) +   // ✅ Correct
  (userFeedback × 0.15) +     // ✅ Correct
  (strainModifier × 0.10)     // ⚠️ Should use full strain formula
```

**Issues**:

1. Tempo component is not phase-aware (should compare actual vs target phase timings)
2. Strain modifier calculation is simplified (should use: `(0.4 × HR rise) + (0.3 × SpO₂ drop) + (0.3 × recovery delay)`)

---

## 🎯 Missing Features Summary

### Critical Gaps (Blockers):

1. **Phase Detection Engine**
   - No motion sensor analysis for concentric/eccentric detection
   - No "pause" state detection
   - No first-movement-after-pause logic

2. **Split Time Tracking**
   - No `concentricTimeMs` calculation
   - No `eccentricTimeMs` calculation
   - No `pauseTimeMs` tracking
   - No TUT (Time Under Tension) calculation

3. **Trainer UI for Split Times**
   - No live split time display (FR-018 requirement)
   - No phase timing breakdown in trainer dashboard

4. **Eccentric-First Support**
   - No toggle in UI
   - No logic to handle eccentric-first exercises

### Important Gaps (Should Fix):

5. **Target Tempo Comparison**
   - No target tempo pattern input
   - No variance calculation (±15% check)
   - No actual vs target comparison

6. **ACIF v2 Phase Data**
   - Phase timing not included in ACIF signals
   - Cannot learn from tempo patterns in federated learning

7. **Strain Calculation**
   - Simplified strain modifier (HR zones only)
   - Should use: `(0.4 × HR rise) + (0.3 × SpO₂ drop) + (0.3 × recovery delay)`

---

## 🔧 Recommended Implementation Plan

### Phase 1: Core Phase Detection

1. Create `phaseDetectionService.ts` to analyze motion data
2. Detect pause → concentric → pause → eccentric cycles
3. Calculate split times for each phase
4. Support eccentric-first toggle

### Phase 2: Tempo Scoring Enhancement

1. Add target tempo pattern input (e.g., "3-1-2-1")
2. Compare actual phase timings to target
3. Calculate variance (±15% tolerance)
4. Update tempo score based on variance

### Phase 3: UI Updates

1. Add split time display to `IntensityDisplay.svelte`
2. Add trainer-specific split time panel
3. Add eccentric-first toggle to workout settings
4. Add target tempo configuration UI

### Phase 4: ACIF v2 Integration

1. Add phase timing data to ACIF Signal Package
2. Include `concentric_avg_ms`, `eccentric_avg_ms`, `tempo_variance` in signals
3. Update federated learning to use tempo patterns

### Phase 5: Strain Calculation Fix

1. Implement full strain formula from spec
2. Track HR rise, SpO₂ drop, recovery delay
3. Update strain modifier calculation

---

## 📝 Files to Review/Update

### Core Services:

- `app/src/lib/services/intensityScoring.ts` - Needs phase-aware tempo calculation
- `app/src/lib/services/strainCalculator.ts` - Needs full strain formula
- **NEW**: `app/src/lib/services/phaseDetectionService.ts` - Needs to be created

### UI Components:

- `app/src/lib/components/intensity/IntensityDisplay.svelte` - Add split time display
- `app/src/lib/components/intensity/IntensityConfig.svelte` - Add target tempo input
- `app/src/lib/components/trainer/RealtimeWorkoutView.svelte` - Add trainer split times
- **NEW**: `app/src/lib/components/intensity/PhaseTimingDisplay.svelte` - Create new component

### Data Models:

- `specs/003-intensity-score-live/data-model.md` - TempoData entity defined but not used
- `specs/030-acif-federated-learning/data-model.md` - ACIF signals need phase data

### Contracts:

- `specs/003-intensity-score-live/contracts/api-endpoints.md` - API may need phase data endpoints

---

## ✅ Next Steps

1. **Review this analysis** with the team
2. **Prioritize gaps** based on user impact
3. **Create implementation tickets** for each phase
4. **Update ACIF v2 spec** to include phase timing data
5. **Design phase detection algorithm** for motion sensors

---

## Questions for Clarification

1. **Motion Sensors**: Which sensors are available? (Accelerometer, Gyroscope, Magnetometer?)
2. **Phase Detection Accuracy**: What's the acceptable margin of error for phase detection?
3. **Target Tempo Input**: Should users input target tempo per exercise, or use defaults?
4. **Trainer Split Times**: Real-time only, or also historical view?
5. **ACIF Privacy**: Should phase timing data be anonymized differently than other signals?

---

**Generated**: 2025-11-05
**Analysis by**: AI Assistant
**Status**: Ready for Review

## Related

^[{src_rel}]
