---
title: ACIF v2 — Phase Tracking and Intensity Integration
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/ACIF v2 — Phase Tracking and Intensity Integration.md"]
updated: 2026-07-24
---

# ACIF v2 — Phase Tracking and Intensity Integration

> NOTE: This is a canonical doctrine note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

---

## Purpose

Define the enhanced ACIF v2 signal schema that consolidates health metrics through the intensity score, adds phase timing (concentric/eccentric/TUT), superset membership and unilateral side signals, and optional nutrition and supplement tracking for adaptive AI learning.

---

## Core Principle

Raw health metrics (HR, SpO2) are not included in ACIF signals. The intensity score already aggregates them — the federated learning model learns from processed metrics, not raw sensor data. This reduces duplication and improves privacy.

---

## Definitions

- **ACIF (Adaptive Coaching Intelligence Feedback)** — the signal package sent to the federated learning pipeline after each workout session
- **Intensity score** — a 0–100 composite that consolidates HR, SpO2, recovery, and movement quality
- **Phase timing** — concentric, eccentric, and pause durations per rep, from motion sensor analysis
- **TUT (Time Under Tension)** — total phase time for a set
- **Stack hash** — anonymized identifier for a supplement combination; no product names

---

## System Structure

### ACIF v2 Signal Schema

```typescript
interface ACIFSignalPackage {
  app: "fit" | "mind" | "echo";
  mode: "workout" | "journal" | "behavior";
  timestamp: string; // ISO 8601

  signals: {
    // INTENSITY (consolidated health metric)
    intensity_score: number;           // 0–100
    intensity_breakdown: {
      tempo: number;
      smoothness: number;
      consistency: number;
      strain_modifier: number;         // 0.85 | 0.95 | 1.0
    };

    // PHASE TIMING (resistance training only)
    phase_timing?: {
      concentric_avg_ms: number;
      eccentric_avg_ms: number;
      pause_avg_ms: number;
      total_tut_ms: number;
      tempo_variance: number;          // 0–100, lower is better
      is_eccentric_first: boolean;
    };

    // MUSIC CORRELATION
    music_bpm?: number;
    music_genre?: string;
    music_energy?: number;             // 0–100

    // NUTRITION (optional, privacy-respecting)
    nutrition?: {
      pre_workout_calories?: number;
      pre_workout_carbs?: number;
      pre_workout_protein?: number;
      hydration_ml?: number;
      caffeine_mg?: number;
      time_since_meal_mins?: number;
    };

    // SUPPLEMENT TRACKING (optional, anonymized)
    supplement_stack?: {
      stack_hash: string;
      days_on_stack: number;
      stack_changed: boolean;
      categories: string[];            // e.g. ["pre-workout", "creatine"]
    };

    // CONTEXT & MOOD
    mood_predicted: number;            // 1–5 (physiological signals)
    fatigue?: number;                  // 1–5 (reported or predicted)
    location_category: string;

    // EXERCISE CONTEXT
    reps_delta?: number;
    volume_delta?: number;
    rest_time_ms?: number;

    // SUPERSET / UNILATERAL (optional)
    supersetGroupId?: string;           // links sets belonging to the same superset
    supersetOrder?: number;             // position within the superset (1-based)
    isUnilateral?: boolean;             // true for single-limb exercises
    side?: "left" | "right" | "both" | "unknown";
  };
}
```

---

## Rules

- **Never include raw HR or SpO2** in ACIF signals — route through intensity score only
- Supplement tracking must use a `stack_hash` — no product names, no Rx compounds
- Nutrition fields are all optional — never required
- Phase timing is only present for resistance training sessions
- `tempo_variance` is deviation from target tempo pattern (±15% acceptable per Spec 003)

---

## Flow

```text
Motion sensors → Phase Detection Service → phase_timing fields
Wearable (HR/SpO2/HRV) → Strain Calculator → intensity_breakdown.strain_modifier → intensity_score
Nutrition input → Nutrition Preprocessor → nutrition fields
User supplement log → Stack anonymizer (hash) → supplement_stack fields
All fields → ACIFSignalPackage → Federated Learning Pipeline
```

---

## Relationships

See also: [[EVOtraining — Exercise Data Gap Analysis]], [[EVOtraining — StrainSync System]], [[Exercise-End Intensity Scoring]]

---

## Edge Cases / Special Handling

- Phase timing is absent for cardio/flexibility/mobility sessions — schema allows `phase_timing` to be omitted
- Supplement categories must be generic (no brand names); hash must not be reversible to product identity
- Nutrition data is captured only when user has opted into nutrition tracking

---

## Summary

ACIF v2 replaces raw HR/SpO2 fields with a consolidated `intensity_score` and `intensity_breakdown`. New fields add phase timing (TUT, concentric/eccentric averages, tempo variance), superset membership (`supersetGroupId`, `supersetOrder`) and unilateral side signals (`isUnilateral`, `side`) — see [[EVOtraining — Exercise Data Gap Analysis]] — and optional nutrition/supplement tracking. The federated pipeline learns from processed metrics, not raw sensor values.

## Related

^[{src_rel}]
