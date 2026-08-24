---
title: EVOtraining — Exercise Data Gap Analysis
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVOtraining — Exercise Data Gap Analysis.md
updated: 2026-07-24
---

# EVOtraining — Exercise Data Gap Analysis

> NOTE: This is a canonical doctrine note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

---

## Purpose

Document the known gaps in EVOtraining's exercise data model relative to spec requirements. Covers concentric/eccentric phase tracking, superset grouping, and unilateral exercise tracking — all absent from the current implementation as of the gap analysis date (2025-11-05).

---

## Core Principle

Phase timing, superset structure, and unilateral side data are first-class signals for the EVOtraining adaptive model. They cannot be approximated from aggregate intensity or rep counts alone.

---

## Definitions

- **Phase detection** — identifying concentric (lifting), eccentric (lowering), and pause states from motion sensor data per rep
- **TUT (Time Under Tension)** — sum of concentric + eccentric + pause time per rep
- **Superset** — two or more exercises performed back-to-back with zero rest between them, rest only after the full group
- **Unilateral exercise** — single-limb movement (e.g., Bulgarian split squat, single-arm row) where left/right sides may differ
- **Bilateral deficit** — when combined single-limb output is less than bilateral output

---

## System Structure

### Current Intensity Formula (partial implementation)

```
overall = (tempo × 0.30) + (motionSmoothness × 0.25) + (repConsistency × 0.20) + (userFeedback × 0.15) + (strainModifier × 0.10)
```

- Tempo component uses heart rate patterns — NOT phase-aware
- Strain modifier is simplified (HR zones only, not full formula)

### Specified Strain Formula (from Spec 003)

```
strainModifier = (0.4 × HR_rise) + (0.3 × SpO2_drop) + (0.3 × recovery_delay)
```

---

## Rules

### Phase tracking requirements (Spec 003)

- FR-016: Identify first movement after pause as concentric, reverse as eccentric
- FR-017: Score tempo within ±15% of target values
- FR-018: Trainers must see live split times for eccentric/concentric phases
- FR-019: Provide toggle for eccentric-first movement patterns (e.g., Romanian Deadlifts)

### Superset rules

- Zero rest between exercises within a superset
- Fatigue multiplier applies to the second exercise: base score adjusted upward (no recovery time)
- Overall superset intensity = average of all exercises in group (after fatigue adjustment)

### Unilateral rules

- Track intensity per side independently
- Use the limiting (weaker) side for overall score
- Flag imbalances >10% for corrective recommendation

---

## Flow

### Missing implementation paths

1. **Phase detection** → `phaseDetectionService.ts` (not yet created)
   - Input: motion sensor data (accelerometer / gyroscope)
   - Output: `{ phase: 'concentric'|'eccentric'|'pause', timestamp, duration }`

2. **Split time tracking** → `concentricTimeMs`, `eccentricTimeMs`, `pauseTimeMs` per rep
   - Currently absent from `SetCompletion` schema

3. **Superset grouping** → `WorkoutExercise.grouping` field (not in current schema)
   - Missing: `type`, `groupId`, `orderInGroup`, `restBetweenExercises`, `restBetweenRounds`

4. **Unilateral side data** → `SetCompletion.sideData` (not in current schema)
   - Missing: per-side reps/weight/RPE/phaseTiming, `imbalance.strengthDifference`

---

## Relationships

See also: [[Exercise-End Intensity Scoring]], [[EVOtraining — StrainSync System]], [[ACIF v2 — Phase Tracking and Intensity Integration]]

---

## Edge Cases / Special Handling

- **Eccentric-first exercises** (e.g., Romanian Deadlifts): toggle must reverse phase-detection logic
- **Circuits vs supersets**: optional 10–20s between exercises for circuits (vs strict 0s for supersets); rest between rounds still applies
- **Giant sets** (4+ exercises, 0 rest): treated the same as supersets but accumulate fatigue across more exercises
- **Bilateral deficit**: when tracking unilateral, combined single-limb output may differ from bilateral — do not conflate

---

## Summary

Three major data gaps exist in EVOtraining: (1) phase detection and TUT tracking per rep, (2) superset grouping with zero-rest fatigue accounting, and (3) unilateral side-by-side tracking with imbalance detection. All three feed into the ACIF signal package and the adaptive training model. Phase detection requires a new `phaseDetectionService.ts`; superset and unilateral require schema additions to `WorkoutExercise` and `SetCompletion`.

## Related

^[source-materials/mirrors/doctrine/EVOtraining — Exercise Data Gap Analysis.md]
