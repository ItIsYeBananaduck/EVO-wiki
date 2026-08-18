---
title: CURRENT_TRAINING_DATA_STATUS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CURRENT_TRAINING_DATA_STATUS.md"]
updated: 2026-07-24
---

# Current Training Data Status

## Question: Are we set up for set-level training with StrainSync?

## Answer: **PARTIALLY** - Infrastructure exists, but transformation code is missing

---

## What EXISTS ✅

### 1. Set-Level Music Logging ✅

- **Status**: **FULLY IMPLEMENTED**
- **Location**: `flutter_app/lib/features/intensity/domain/strain_sync_music_log_service.dart`
- **Model**: `StrainSyncSetMusicLog` with:
  - ✅ `exerciseId` - Matches to specific exercise
  - ✅ `setIndex` - Matches to specific set (0, 1, 2, ...)
  - ✅ `trackKey`, `trackName`, `artistName`
  - ✅ `phase` (warmup/main/peak/cooldown)
  - ✅ `avgHeartRate` during that set
- **Logging**: Happens per set in `live_workout_screen.dart` (line 1883-1895)
- **Storage**: SharedPreferences, keyed by `sessionId`

### 2. Set-Level Workout Data ✅

- **Status**: **FULLY IMPLEMENTED**
- **Location**: `flutter_app/lib/features/intensity/domain/intensity_models.dart`
- **Model**: `OnDeviceWorkoutSet` with:
  - ✅ `rpe` (per set)
  - ✅ `reps`, `weight` (per set)
  - ✅ `avgHeartRate` (per set)
  - ✅ `notes` (per set)
- **Storage**: Isar database via `OnDeviceWorkoutLog`

### 3. Taste Preferences ✅

- **Status**: **FULLY IMPLEMENTED**
- **Location**: `flutter_app/lib/features/intensity/domain/strain_sync_track_store.dart`
- **Model**: `StrainSyncTrackMeta` with:
  - ✅ `starred` (taste more)
  - ✅ `skipped` (taste less)
  - ✅ `playCount`

---

## What's MISSING ❌

### 1. WorkoutLogSummary Model ❌

- **Status**: **NOT IMPLEMENTED**
- **Spec Location**: `specs/034-on-device-lora/data-model.md`
- **Expected**: Workout-level summary (not set-level)
- **Fields**: `id`, `recordedAt`, `hrvTrend`, `rpeScore`, `formNotes`, `usedInTraining`
- **Problem**: Spec is workout-level, but we need set-level for StrainSync

### 2. Transformation Service ❌

- **Status**: **DOES NOT EXIST**
- **Required**: Code to:
  1. Query `OnDeviceWorkoutLog` from Isar
  2. Query `StrainSyncSetMusicLog[]` from SharedPreferences
  3. Query `StrainSyncTrackMeta` from SharedPreferences
  4. **Join music logs to sets** by `exerciseId` + `setIndex`
  5. **Create set-level training samples** (one per set)
  6. Format as JSONL for llama.cpp training
- **Location**: Should be in Flutter service layer (doesn't exist)

### 3. Training Data Format Mismatch ⚠️

- **Spec says**: `WorkoutLogSummary` (workout-level)
- **Contract says**: `logBatchIds: string[]` (WorkoutLogSummary IDs)
- **Reality**: We need **set-level** samples to preserve track → performance correlation
- **Action**: Need to update spec/contract OR create set-level transformation

---

## Current Data Flow

### What Happens Now:

```
1. User completes set
   ↓
2. Music is logged: StrainSyncSetMusicLog (setIndex, exerciseId, trackKey)
   ↓
3. Set performance logged: OnDeviceWorkoutSet (rpe, reps, weight)
   ↓
4. Both stored separately (SharedPreferences + Isar)
   ↓
5. ❌ NO CODE to join them for training
```

### What We Need:

```
1. Query workout logs (Isar)
   ↓
2. Query music logs (SharedPreferences) by sessionId
   ↓
3. Query taste preferences (SharedPreferences) by trackKey
   ↓
4. ⚠️ JOIN: Match music logs to sets by exerciseId + setIndex
   ↓
5. ⚠️ CREATE: Set-level training samples (one per set)
   ↓
6. ⚠️ FORMAT: JSONL for llama.cpp training
```

---

## Gap Analysis

### Spec vs. Reality

| Component           | Spec Says                           | Reality                                                             | Gap                       |
| ------------------- | ----------------------------------- | ------------------------------------------------------------------- | ------------------------- |
| **Data Model**      | `WorkoutLogSummary` (workout-level) | `OnDeviceWorkoutLog` (workout) + `OnDeviceWorkoutSet[]` (set-level) | ⚠️ Need set-level model   |
| **Music Data**      | Not explicitly in spec              | `StrainSyncSetMusicLog` (set-level)                                 | ✅ Already set-level      |
| **Training Format** | Workout-level summaries             | Need set-level samples                                              | ❌ Transformation missing |
| **Join Logic**      | Not specified                       | Need exerciseId + setIndex matching                                 | ❌ Code missing           |

---

## What Needs to Be Built

### 1. Set-Level Training Sample Model

```dart
class SetTrainingSample {
  final String id; // "${workoutId}_${exerciseId}_${setIndex}"
  final String workoutId;
  final String exerciseId;
  final String exerciseName;
  final int setIndex;
  final String phase;
  final DateTime recordedAt;

  // Performance for THIS set
  final SetPerformance performance;

  // Music for THIS set (nullable)
  final SetMusic? music;

  // Context (shared across sets)
  final WorkoutContext context;
}

class SetPerformance {
  final int reps;
  final double weight;
  final double? rpe;
  final double? avgHeartRate;
  final double? restSeconds;
}

class SetMusic {
  final String trackKey;
  final String trackName;
  final String artistName;
  final String phase;
  final bool tasteMore;
  final bool tasteLess;
  final int playCount;
}
```

### 2. Transformation Service

```dart
class TrainingDataTransformer {
  Future<List<SetTrainingSample>> buildSetLevelSamples({
    required String userId,
    required LocalDateRange dateRange,
  }) async {
    // 1. Query workout logs
    final workouts = await onDeviceDataStore.getWorkoutLogs(userId, dateRange);

    // 2. Query strain records (for HRV)
    final strainRecords = await onDeviceDataStore.getStrainRecords(userId, dateRange);

    // 3. Query music logs per workout
    final Map<String, List<StrainSyncSetMusicLog>> musicLogsBySession = {};
    for (final workout in workouts) {
      final logs = await strainSyncMusicLogService.loadLogsForSession(
        userId: userId,
        sessionId: workout.id,
      );
      if (logs.isNotEmpty) {
        musicLogsBySession[workout.id] = logs;
      }
    }

    // 4. Query taste preferences
    final Map<String, StrainSyncTrackMeta> trackMeta = {};
    // ... (cache all track metadata)

    // 5. Build set-level samples
    final List<SetTrainingSample> samples = [];
    for (final workout in workouts) {
      final musicLogs = musicLogsBySession[workout.id] ?? [];
      final strainRecord = strainRecords.firstWhere(
        (r) => r.localDate == workout.localDate,
        orElse: () => null,
      );

      // Group music logs by exercise + setIndex
      final Map<String, Map<int, StrainSyncSetMusicLog>> musicByExerciseSet = {};
      for (final musicLog in musicLogs) {
        musicByExerciseSet.putIfAbsent(
          musicLog.exerciseId,
          () => {},
        )[musicLog.setIndex] = musicLog;
      }

      // Create sample for each set
      for (final exercise in workout.exercises) {
        for (int setIndex = 0; setIndex < exercise.sets.length; setIndex++) {
          final set = exercise.sets[setIndex];
          final musicLog = musicByExerciseSet[exercise.exerciseId]?[setIndex];
          final trackMeta = musicLog != null
              ? trackMeta[musicLog.trackKey]
              : null;

          samples.add(SetTrainingSample(
            id: '${workout.id}_${exercise.id}_$setIndex',
            workoutId: workout.id,
            exerciseId: exercise.exerciseId,
            exerciseName: exercise.exerciseName,
            setIndex: setIndex,
            phase: musicLog?.phase ?? 'main',
            recordedAt: set.performedAt ?? workout.startedAt,
            performance: SetPerformance(
              reps: set.reps,
              weight: set.weight,
              rpe: set.rpe,
              avgHeartRate: set.avgHeartRate ?? musicLog?.avgHeartRate,
              restSeconds: set.restSeconds,
            ),
            music: musicLog != null ? SetMusic(
              trackKey: musicLog.trackKey,
              trackName: musicLog.trackName,
              artistName: musicLog.artistName,
              phase: musicLog.phase,
              tasteMore: trackMeta?.starred ?? false,
              tasteLess: trackMeta?.skipped ?? false,
              playCount: trackMeta?.playCount ?? 0,
            ) : null,
            context: WorkoutContext(
              hrvTrend: strainRecord?.hrvScore != null
                  ? [strainRecord!.hrvScore]
                  : [],
              formNotes: _combineNotes(workout, exercise, set),
              workoutDate: workout.localDate,
            ),
          ));
        }
      }
    }

    return samples;
  }
}
```

### 3. JSONL Formatter

```dart
String formatAsTrainingJSONL(List<SetTrainingSample> samples) {
  final List<String> lines = [];
  for (final sample in samples) {
    final json = {
      'id': sample.id,
      'workoutId': sample.workoutId,
      'exerciseName': sample.exerciseName,
      'setIndex': sample.setIndex,
      'phase': sample.phase,
      'recordedAt': sample.recordedAt.toIso8601String(),
      'performance': {
        'reps': sample.performance.reps,
        'weight': sample.performance.weight,
        'rpe': sample.performance.rpe,
        'avgHeartRate': sample.performance.avgHeartRate,
        'restSeconds': sample.performance.restSeconds,
      },
      if (sample.music != null) 'music': {
        'trackKey': sample.music!.trackKey,
        'trackName': sample.music!.trackName,
        'artistName': sample.music!.artistName,
        'phase': sample.music!.phase,
        'tasteMore': sample.music!.tasteMore,
        'tasteLess': sample.music!.tasteLess,
        'playCount': sample.music!.playCount,
      },
      'context': {
        'hrvTrend': sample.context.hrvTrend,
        'formNotes': sample.context.formNotes,
        'workoutDate': sample.context.workoutDate,
      },
    };
    lines.add(jsonEncode(json));
  }
  return lines.join('\n');
}
```

---

## Summary

### ✅ What We Have:

- Set-level music logging (`StrainSyncSetMusicLog` with `setIndex`)
- Set-level workout data (`OnDeviceWorkoutSet` with `rpe`, `reps`, etc.)
- Taste preferences (`StrainSyncTrackMeta`)
- All data sources accessible

### ❌ What We're Missing:

- **Transformation service** to join music logs to sets
- **Set-level training sample model** (currently spec says workout-level)
- **JSONL formatter** for training data
- **Integration** into training pipeline

### ⚠️ Spec Mismatch:

- Spec defines `WorkoutLogSummary` (workout-level)
- But we need **set-level** samples for StrainSync to work
- **Action**: Either update spec OR create set-level transformation that feeds into training

---

## Recommendation

**Option A**: Update spec to support set-level training

- Change `WorkoutLogSummary` to `SetTrainingSample`
- Update contract to accept set-level samples
- Build transformation service as described above

**Option B**: Keep workout-level spec, but create set-level samples internally

- Build set-level samples in transformation
- Aggregate to workout-level for spec compliance (loses granularity)
- **Not recommended** - loses track → performance correlation

**Option C**: Hybrid approach

- Keep `WorkoutLogSummary` for non-StrainSync workouts
- Add `SetTrainingSample[]` for StrainSync-enabled workouts
- Training pipeline handles both formats

**Recommended**: **Option A** - Update spec to set-level, as that's what StrainSync requires.

---

## Next Steps

1. **Update spec** (`specs/034-on-device-lora/data-model.md`) to include set-level training samples
2. **Create transformation service** (`TrainingDataTransformer`)
3. **Create set-level models** (`SetTrainingSample`, `SetPerformance`, `SetMusic`)
4. **Build JSONL formatter**
5. **Integrate** into training pipeline

**We have the infrastructure, we just need to wire it together!**

## Related

^[{src_rel}]
