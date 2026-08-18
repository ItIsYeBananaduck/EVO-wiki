---
title: STRAINSYNC_TRAINING_DATA
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/STRAINSYNC_TRAINING_DATA.md"]
updated: 2026-07-24
---

# StrainSync Music Data in Training

## Overview

**StrainSync** music data is logged separately from workout logs but should be included in training data. This document explains how to access and integrate it.

## Data Storage

### 1. Music Logs (Per Set)

- **Storage**: SharedPreferences (not Isar)
- **Service**: `StrainSyncMusicLogService`
- **Key Pattern**: `strainsync_set_logs_${userId}_$sessionId`
- **Model**: `StrainSyncSetMusicLog`

**Fields**:

- `userId`, `sessionId`
- `exerciseId`, `exerciseName`
- `setIndex` (which set in the exercise)
- `phase` (warmup/main/peak/cooldown)
- `trackKey` (e.g., "apple_music:trackId")
- `trackName`, `artistName`
- `playlistId` (optional)
- `avgHeartRate` (during music playback)
- `loggedAt` (timestamp)

**Access**:

```dart
final List<StrainSyncSetMusicLog> musicLogs =
    await strainSyncMusicLogService.loadLogsForSession(
      userId: userId,
      sessionId: workoutLog.id, // Match workout log ID
    );
```

### 2. Taste Preferences (Per Track)

- **Storage**: SharedPreferences
- **Service**: `StrainSyncTrackStore`
- **Key Pattern**: `strainsync_track_meta_$trackKey`
- **Model**: `StrainSyncTrackMeta`

**Fields**:

- `trackKey` (provider-qualified, e.g., "apple_music:trackId")
- `starred` (boolean - "taste more")
- `skipped` (boolean - "taste less")
- `playCount` (number of times played)
- `lastPlayedAt` (timestamp)

**Access**:

```dart
final StrainSyncTrackMeta meta =
    await strainSyncTrackStore.getOrCreate(trackKey);
```

## Integration with WorkoutLogSummary

**IMPORTANT**: StrainSync tracks music **per set**, not per phase. The training data must preserve set-level granularity to learn how specific tracks affect performance.

### Set-Level Data Structure

Each set should include:

- **Track info**: What song was playing
- **Performance metrics**: RPE, reps, weight, heart rate for that set
- **Taste preference**: Did user "taste more" (star) or "taste less" (skip) this track
- **Phase context**: Which phase (warmup/main/peak/cooldown) the set was in

### Training Data Format

The training data should be **set-level**, not workout-level:

```json
{
  "id": "set-uuid",
  "workoutId": "workout-uuid",
  "exerciseName": "Bench Press",
  "setIndex": 2,
  "phase": "main",
  "recordedAt": "2025-11-22T14:35:00Z",
  "performance": {
    "reps": 8,
    "weight": 100,
    "rpe": 7,
    "avgHeartRate": 145,
    "restSeconds": 120
  },
  "music": {
    "trackKey": "apple_music:trackId123",
    "trackName": "Eye of the Tiger",
    "artistName": "Survivor",
    "tasteMore": true, // User starred this track
    "tasteLess": false,
    "playCount": 5
  },
  "hrvTrend": [45.2, 48.1, 46.5], // HRV context for this workout
  "formNotes": "Felt strong, good form"
}
```

This allows the model to learn:

- "When Track X plays, user tends to rate RPE higher"
- "Track Y correlates with more reps at same weight"
- "User prefers Track Z during peak phase"

## Transformation Logic

### Step 1: Query All Data Sources

```dart
// 1. Get workout logs
final List<OnDeviceWorkoutLog> workouts =
    await onDeviceDataStore.getWorkoutLogs(userId, dateRange);

// 2. Get strain records (for HRV context)
final List<OnDeviceStrainRecord> strainRecords =
    await onDeviceDataStore.getStrainRecords(userId, dateRange);

// 3. For each workout, get music logs (per set)
final Map<String, List<StrainSyncSetMusicLog>> musicLogsBySession = {};
for (final workout in workouts) {
  final List<StrainSyncSetMusicLog> logs =
      await strainSyncMusicLogService.loadLogsForSession(
        userId: userId,
        sessionId: workout.id,
      );
  if (logs.isNotEmpty) {
    musicLogsBySession[workout.id] = logs;
  }
}

// 4. Get taste preferences for all tracks
final Map<String, StrainSyncTrackMeta> trackMeta = {};
for (final logs in musicLogsBySession.values) {
  for (final log in logs) {
    if (!trackMeta.containsKey(log.trackKey)) {
      trackMeta[log.trackKey] =
          await strainSyncTrackStore.getOrCreate(log.trackKey);
    }
  }
}
```

### Step 2: Build Set-Level Training Samples

**CRITICAL**: Preserve set-level granularity to learn track → performance correlations.

```dart
List<SetTrainingSample> buildSetLevelSamples(
  OnDeviceWorkoutLog workout,
  OnDeviceStrainRecord? strainRecord,
  List<StrainSyncSetMusicLog>? musicLogs,
  Map<String, StrainSyncTrackMeta> trackMeta,
) {
  final List<SetTrainingSample> samples = [];

  // Get HRV context for this workout
  final double? hrvScore = strainRecord?.hrvScore;
  final List<double> hrvTrend = hrvScore != null ? [hrvScore] : [];

  // Build a map of music logs by exercise + setIndex
  final Map<String, Map<int, StrainSyncSetMusicLog>> musicByExerciseSet = {};
  if (musicLogs != null) {
    for (final musicLog in musicLogs) {
      musicByExerciseSet.putIfAbsent(
        musicLog.exerciseId,
        () => {},
      )[musicLog.setIndex] = musicLog;
    }
  }

  // Create a training sample for each set
  for (final exercise in workout.exercises) {
    for (int setIndex = 0; setIndex < exercise.sets.length; setIndex++) {
      final set = exercise.sets[setIndex];

      // Find matching music log for this set
      final musicLog = musicByExerciseSet[exercise.exerciseId]?[setIndex];
      final trackMeta = musicLog != null
          ? trackMeta[musicLog.trackKey]
          : null;

      // Build set-level sample
      samples.add(SetTrainingSample(
        id: '${workout.id}_${exercise.id}_$setIndex',
        workoutId: workout.id,
        exerciseName: exercise.exerciseName,
        exerciseId: exercise.exerciseId,
        setIndex: setIndex,
        phase: musicLog?.phase ?? 'main', // Default to main if no music
        recordedAt: set.performedAt ?? workout.startedAt,

        // Performance metrics for THIS set
        performance: SetPerformance(
          reps: set.reps,
          weight: set.weight,
          rpe: set.rpe,
          avgHeartRate: set.avgHeartRate ?? musicLog?.avgHeartRate,
          restSeconds: set.restSeconds,
          durationSeconds: set.durationSeconds,
        ),

        // Music data for THIS set
        music: musicLog != null ? SetMusic(
          trackKey: musicLog.trackKey,
          trackName: musicLog.trackName,
          artistName: musicLog.artistName,
          phase: musicLog.phase,
          tasteMore: trackMeta?.starred ?? false,
          tasteLess: trackMeta?.skipped ?? false,
          playCount: trackMeta?.playCount ?? 0,
        ) : null,

        // Context (shared across sets in workout)
        context: WorkoutContext(
          hrvTrend: hrvTrend,
          formNotes: _combineNotes(workout, exercise, set),
          workoutDate: workout.localDate,
        ),
      ));
    }
  }

  return samples;
}

String _combineNotes(
  OnDeviceWorkoutLog workout,
  OnDeviceWorkoutExercise exercise,
  OnDeviceWorkoutSet set,
) {
  final List<String> notes = [];
  if (workout.notes != null && workout.notes!.isNotEmpty) {
    notes.add(workout.notes!);
  }
  if (exercise.notes != null && exercise.notes!.isNotEmpty) {
    notes.add(exercise.notes!);
  }
  if (set.notes != null && set.notes!.isNotEmpty) {
    notes.add(set.notes!);
  }
  return notes.join(' ').substring(0, min(512, notes.join(' ').length));
}
```

## Training Data Format

The final training JSONL should be **set-level**, with one entry per set:

```json
{
  "id": "workout-uuid_exercise-uuid_0",
  "workoutId": "workout-uuid",
  "exerciseName": "Bench Press",
  "exerciseId": "exercise-uuid",
  "setIndex": 0,
  "phase": "main",
  "recordedAt": "2025-11-22T14:30:00Z",
  "performance": {
    "reps": 10,
    "weight": 100,
    "rpe": 7,
    "avgHeartRate": 145,
    "restSeconds": 120,
    "durationSeconds": 45
  },
  "music": {
    "trackKey": "apple_music:trackId123",
    "trackName": "Eye of the Tiger",
    "artistName": "Survivor",
    "phase": "main",
    "tasteMore": true,
    "tasteLess": false,
    "playCount": 5
  },
  "context": {
    "hrvTrend": [45.2, 48.1, 46.5],
    "formNotes": "Felt strong, good form",
    "workoutDate": "2025-11-22"
  }
}
```

**Multiple sets from same workout**:

```json
// Set 1
{"id": "workout-uuid_exercise-uuid_0", "setIndex": 0, "music": {"trackKey": "trackA"}, "performance": {"rpe": 7, "reps": 10}}
// Set 2
{"id": "workout-uuid_exercise-uuid_1", "setIndex": 1, "music": {"trackKey": "trackB"}, "performance": {"rpe": 8, "reps": 9}}
// Set 3
{"id": "workout-uuid_exercise-uuid_2", "setIndex": 2, "music": {"trackKey": "trackC"}, "performance": {"rpe": 6, "reps": 11}}
```

This allows the model to learn:

- **Track A** → RPE 7, 10 reps
- **Track B** → RPE 8, 9 reps (worse performance?)
- **Track C** → RPE 6, 11 reps (better performance!)

The model can then learn: "When Track C plays, user performs better (lower RPE, more reps)"

## Key Points

1. **Music data is separate**: Not in Isar, stored in SharedPreferences
2. **Join by sessionId**: Match `StrainSyncSetMusicLog.sessionId` with `OnDeviceWorkoutLog.id`
3. **Join by exercise + setIndex**: Match `StrainSyncSetMusicLog.exerciseId` + `setIndex` with `OnDeviceWorkoutExercise.exerciseId` + set position
4. **Taste preferences**: Join `StrainSyncSetMusicLog.trackKey` with `StrainSyncTrackMeta.trackKey`
5. **PRESERVE SET-LEVEL GRANULARITY**: Each set gets its own training sample with its specific track and performance
6. **Learn correlations**: Model learns "Track X → better performance" or "Track Y → higher RPE"

## Implementation Notes

- Music logs are session-scoped, so query per workout session
- **Match music logs to sets** by `exerciseId` + `setIndex` (not just sessionId)
- Taste preferences are track-scoped, cache them to avoid repeated queries
- If no music data exists for a set, `music` field should be `null` (still include the set for training)
- **Each set = one training sample** with its specific track and performance metrics
- The model learns: "When this track plays during this exercise, user tends to perform X"

## Why Set-Level Matters

**Example Learning**:

- Set 1: Track "Eye of the Tiger" → RPE 7, 10 reps
- Set 2: Track "Chill Vibes" → RPE 8, 9 reps
- Set 3: Track "Eye of the Tiger" again → RPE 6, 11 reps

The model learns: "Eye of the Tiger" correlates with better performance (lower RPE, more reps) for this user during bench press.

If we aggregated by phase, we'd lose this set-level correlation and couldn't learn which specific tracks help performance.

## Related

^[{src_rel}]
