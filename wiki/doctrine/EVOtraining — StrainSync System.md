---
title: EVOtraining — StrainSync System
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOtraining — StrainSync System.md"]
updated: 2026-07-24
---

# EVOtraining — StrainSync System
> NOTE: This is a canonical doctrine note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

> RULE: All `related` entries must use Obsidian wiki link format.

---

## Purpose
Define how Alice learns how music affects user performance and uses that information to curate adaptive workout playlists.

StrainSync enables:
- music selection per set
- performance correlation with music
- personalized playlist generation

---

## Core Principle
StrainSync learns **correlations between music and performance**, not causation.

It supports performance but does not define it.

---

## Definitions

**Strain (Metric)**
A physiological or perceived difficulty signal.

**StrainSync**
A system that logs music per set and learns how music correlates with performance.

**Set-Level Music Log**
A record of what track was playing during a specific set.

**Performance Metrics**
Reps, weight, RPE, heart rate, tempo, and completion.

**Music Preference**
User feedback such as:
- taste more (star)
- taste less (skip)
- play frequency

---

## System Structure

StrainSync operates at **set-level granularity**:

1. Input
   - workout logs (sets)
   - music logs (per set)
   - taste preferences

2. Join Layer
   - match music to sets via:
     - sessionId
     - exerciseId
     - setIndex

3. Pattern Detection
   - track → performance correlations
   - genre → performance trends
   - phase-based preferences

4. Output
   - playlist adaptation
   - track recommendations

---

## Rules

- StrainSync must NOT be confused with strain metrics
- StrainSync must NOT directly modify:
  - load
  - reps
  - sets

- StrainSync must preserve set-level granularity
- Music data must always be linked to performance context

- User preference overrides inferred performance correlations

---

## Flow

Workout Execution
→ Set Logs
→ Music Logs (per set)
→ Journal Interpretation
→ Pattern Detection (music ↔ performance)
→ Playlist Adaptation

---

## Relationships

(Defined in frontmatter)

---

## Edge Cases / Special Handling

### Correlation vs Causation

If:
- performance improves with a track

Alice must:
- treat it as correlation
- not assume the track caused improvement

---

### Conflicting Signals

If:
- Track A → better performance
- User prefers Track B

Then:
- prioritize user preference
- optionally suggest Track A

---

### Cold Start

If no data exists:

- use user preferences
- fallback to general high-performance playlists

---

### Phase Awareness

Music may adapt based on:

- warmup
- main
- peak
- cooldown

Patterns must be evaluated per phase.

---

## Role in Adapter Training

StrainSync data **is part of training**, but only for music adaptation.

Training uses:

- set-level performance
- associated music
- user preference signals

StrainSync must NOT:

- influence general workout adaptation
- override training logic
- define fatigue or strain

---

## Separation from Training System

StrainSync contributes to:

- music-specific pattern learning

StrainSync does NOT contribute to:

- strength progression
- fatigue modeling
- recovery adaptation

---

## Implementation Reference

### Storage (Flutter / Dart)

| Data | Storage | Service | Key Pattern |
|---|---|---|---|
| Music logs (per set) | SharedPreferences | `StrainSyncMusicLogService` | `strainsync_set_logs_{userId}_{sessionId}` |
| Taste preferences (per track) | SharedPreferences | `StrainSyncTrackStore` | `strainsync_track_meta_{trackKey}` |

Music logs use `sessionId` (matches `OnDeviceWorkoutLog.id`) + `exerciseId` + `setIndex` as the join key.

### Set-Level Training Sample Fields

Each training sample maps one set → one JSONL entry:

```json
{
  "id": "{workoutId}_{exerciseId}_{setIndex}",
  "phase": "warmup|main|peak|cooldown",
  "performance": {
    "reps": 10,
    "weight": 135,
    "rpe": 8,
    "avgHeartRate": 142,
    "restSeconds": 90
  },
  "music": {
    "trackKey": "apple_music:trackId123",
    "tasteMore": false,
    "tasteLess": false,
    "playCount": 1
  },
  "context": {
    "hrvTrend": "stable",
    "workoutDate": "2026-05-05"
  }
}
```

`trackKey` format: `"{provider}:{trackId}"` (e.g., `"apple_music:trackId123"`)

Sets without music data include `"music": null` — still emitted as training samples.

---

## Summary

StrainSync:

- logs music per set
- links music to performance
- learns correlations
- adapts playlists

It is:

- a specialized training input
- scoped to music adaptation only
- separate from physiological strain and workout progression

---

Related notes: [[STRAINSYNC_TRAINING_DATA]]

## Related

^[{src_rel}]
