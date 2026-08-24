---
title: SPEC-monday-workout-integration
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/SPEC-monday-workout-integration.md"]
updated: 2026-07-24
---

# Spec-Kit: Monday Workout Data Integration

**Status**: Draft
**Authors**: GitHub Copilot
**Created**: 2025-09-26
**Updated**: 2025-09-26

## Summary

Integrate Monday intensity analysis and volume adjustment system into the existing git-fit workout tracking infrastructure without disrupting current functionality.

## Motivation

The current workout system lacks weekly intensity analysis and automated volume progression. Users need:

- Weekly workout intensity scoring (0-100%)
- Automated volume adjustments based on performance
- Health data integration (HR, SpO2, sleep)
- Secure data hashing for backend storage

## Goals

**Primary Goals**:

- [ ] Extend existing `WorkoutExecutionService` with Monday data collection
- [ ] Add intensity scoring algorithm (HR variance + SpO2 drift + sleep + user feedback)
- [ ] Implement SHA-256 hashing for secure data transmission
- [ ] Create automated volume adjustment rules (±2.5-10 lbs based on intensity)

**Secondary Goals**:

- [ ] UI integration with existing workout interface
- [ ] Backend storage through Supabase API
- [ ] Weekly processing notifications

## Non-Goals

- Creating separate workout tracking system
- Modifying existing workout completion flow
- Breaking current UI/UX patterns

## Design

### Architecture

```
Existing: WorkoutExecutionService
    ↓
Extended: WorkoutExecutionService + MondayDataCollector
    ↓
New: IntensityCalculator + VolumeAdjuster
    ↓
Backend: Supabase API (workouts Edge Function)
```

### Data Flow

1. **Workout Execution** → Existing `WorkoutExecutionService.completeSet()`
2. **Monday Collection** → `MondayDataCollector.logWorkoutData()`
3. **Intensity Calculation** → `IntensityCalculator.calculateScore()`
4. **Volume Adjustment** → `VolumeAdjuster.getNextWeekVolume()`
5. **Secure Storage** → SHA-256 hash → Supabase backend

### Interface Specifications

#### Extended WorkoutExecutionService

```typescript
interface MondayWorkoutData {
  exerciseId: string;
  reps: number;
  sets: number;
  weight: number;
  workoutTime: number;
  userFeedback: "easy killer" | "good pump" | "struggle city";
  healthData: {
    heartRate: { avg: number; max: number; variance: number };
    spo2: { avgSpO2: number; drift: number };
    sleepScore: number; // 0-20 scale
  };
  timestamp: Date;
}

interface IntensityScore {
  total: number; // 0-100%
  breakdown: {
    base: 50;
    hrVariance: number; // 0-20
    spo2Drift: number; // 0-10
    sleepScore: number; // 0-20
    userFeedback: number; // -15 to +20
  };
}
```

#### Volume Adjustment Rules

```typescript
interface VolumeAdjustment {
  currentWeight: number;
  nextWeight: number;
  adjustment: number;
  reason: string;
  rule: "low_intensity" | "moderate_intensity" | "high_intensity";
}

// Rules:
// < 40% intensity → -5 to -10 lbs
// 40-60% intensity → -2.5 to +2.5 lbs
// > 60% intensity → +2.5 to +5 lbs
```

## Implementation Plan

### Phase 1: Service Extension (Week 1)

- [ ] Extend `WorkoutExecutionService` with Monday data collection
- [ ] Create `IntensityCalculator` service
- [ ] Create `VolumeAdjuster` service
- [ ] Add unit tests for calculation logic

### Phase 2: UI Integration (Week 1)

- [ ] Add user feedback buttons to existing workout interface
- [ ] Add Monday notification system
- [ ] Integrate with existing workout completion flow

### Phase 3: Backend Integration (Week 2)

- [ ] Extend Supabase `workouts` Edge Function with Monday data storage
- [ ] Implement SHA-256 hashing for secure transmission
- [ ] Add Monday processing endpoints

### Phase 4: Testing & Validation (Week 2)

- [ ] Test intensity calculation with provided test case
- [ ] Validate volume adjustments
- [ ] Ensure no conflicts with existing functionality

## Test Cases

### Test Case 1: Basic Intensity Calculation

```
Input: 5 reps, 100 lbs, HR avg 145, "easy killer", 8h sleep
Expected: 47% intensity, +2.5 lbs adjustment
```

### Test Case 2: High Intensity Workout

```
Input: 3 reps, 135 lbs, HR avg 175, "struggle city", 6h sleep
Expected: 25% intensity, -5 lbs adjustment
```

## Risks & Mitigation

**Risk**: Conflicts with existing workout system
**Mitigation**: Extend rather than replace existing services

**Risk**: Performance impact from health data processing
**Mitigation**: Asynchronous processing, cached calculations

**Risk**: Security concerns with health data
**Mitigation**: SHA-256 hashing, encrypted transmission

## Alternatives Considered

1. **Separate Monday System**: Rejected - creates duplicate tracking
2. **Client-side Processing**: Rejected - security concerns
3. **Real-time Adjustments**: Rejected - complexity vs benefit

## Success Metrics

- [ ] Monday data collection working without breaking existing flows
- [ ] Intensity calculations match test cases within 2%
- [ ] Volume adjustments applied correctly
- [ ] Zero conflicts with current workout functionality
- [ ] User feedback integrated seamlessly

## Dependencies

- Existing `WorkoutExecutionService`
- Supabase backend API
- Health data sources (HealthKit/Google Fit)
- SHA-256 hashing library

## Timeline

**Week 1**: Service extension + UI integration
**Week 2**: Backend integration + testing
**Launch**: October 1-8, 2025 (Beta)

---

**Approval**: Pending review and feedback

## Related
