---
title: USER_LORA_TRAINING_IMPLEMENTATION
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/USER_LORA_TRAINING_IMPLEMENTATION.md
updated: 2026-07-24
---

# User LoRA Training Implementation - Complete

## Overview

The User LoRA training pipeline is now fully implemented, providing set-level training data with StrainSync music correlation.

## Architecture

### Data Flow

```
1. Workout Logs (Isar) + Music Logs (SharedPreferences) + Taste Preferences
   ↓
2. TrainingDataTransformer.buildSetLevelSamples()
   - Joins music logs to sets by exerciseId + setIndex
   - Creates one training sample per set
   - Includes track info + performance metrics
   ↓
3. JSONL Formatter
   - Converts samples to JSONL format
   - Writes to temporary file
   ↓
4. TrainingControllerPlugin (Flutter)
   - Calls native TrainingController
   ↓
5. TrainingController.swift (iOS)
   - Validates assets
   - Checks device constraints (battery, charging)
   - Runs llama.cpp training
   - Converts adapter to GGUF
   ↓
6. User LoRA adapter updated
```

## Components

### 1. Set-Level Training Models ✅

**File**: `flutter_app/lib/features/alice/domain/training/set_training_sample.dart`

- `SetTrainingSample`: One sample per set
- `SetPerformance`: Performance metrics (reps, weight, RPE, HR)
- `SetMusic`: Music data (track, artist, phase, taste preferences)
- `WorkoutContext`: Shared context (HRV, notes, date)

### 2. Training Data Transformer ✅

**File**: `flutter_app/lib/features/alice/domain/training/training_data_transformer.dart`

- Queries workout logs from Isar
- Queries music logs from SharedPreferences
- Queries taste preferences from SharedPreferences
- **Joins music logs to sets** by `exerciseId` + `setIndex`
- Creates set-level training samples
- Formats as JSONL

### 3. User LoRA Training Service ✅

**File**: `flutter_app/lib/features/alice/domain/training/user_lora_training_service.dart`

- Orchestrates the full training pipeline
- Prepares training data
- Writes to temporary file
- Calls native training controller
- Handles cleanup

### 4. Training Controller Plugin (Flutter) ✅

**File**: `flutter_app/lib/features/alice/domain/training/training_controller_plugin.dart`

- Flutter method channel wrapper
- Provides type-safe API for:
  - `hasAssets()`: Check required files
  - `runNightlyTraining()`: Execute training
  - `scheduleBackgroundTasks()`: Schedule BG tasks
  - `getStatus()`: Get training status

### 5. Training Controller (iOS) ✅

**File**: `flutter_app/ios/Runner/TrainingController.swift`

- Native Swift plugin
- Validates assets (base model, blank LoRA, user LoRA)
- Checks device constraints (battery, charging, network)
- Runs llama.cpp training (placeholder for full implementation)
- Manages background tasks
- Returns training reports

### 6. AppDelegate Integration ✅

**File**: `flutter_app/ios/Runner/AppDelegate.swift`

- Registers TrainingController method channel
- Wired up at app launch

## Training Data Format

Each training sample is a JSON object with:

```json
{
  "id": "workout-uuid_exercise-uuid_0",
  "workoutId": "workout-uuid",
  "exerciseId": "exercise-uuid",
  "exerciseName": "Bench Press",
  "setIndex": 0,
  "phase": "main",
  "recordedAt": "2025-01-15T10:30:00Z",
  "performance": {
    "reps": 10,
    "weight": 100.0,
    "rpe": 7.5,
    "avgHeartRate": 145.0,
    "restSeconds": 120.0
  },
  "music": {
    "trackKey": "apple_music:track123",
    "trackName": "Eye of the Tiger",
    "artistName": "Survivor",
    "phase": "main",
    "tasteMore": true,
    "tasteLess": false,
    "playCount": 5
  },
  "context": {
    "hrvTrend": [45.2],
    "formNotes": "Felt strong today",
    "workoutDate": "2025-01-15"
  }
}
```

## Usage

### From Flutter

```dart
// Initialize service
final trainingService = UserLoRATrainingService(
  dataStore: isarOnDeviceDataStore,
  musicLogService: strainSyncMusicLogService,
  trackStore: strainSyncTrackStore,
);

// Run nightly training
final report = await trainingService.runNightlyTraining(
  userId: user.id,
  daysBack: 7,
  stepCount: 100,
);

// Check status
final status = await trainingService.getStatus();
```

### Background Tasks

```dart
// Schedule background tasks (call on app launch)
await trainingService.scheduleBackgroundTasks();
```

## Remaining Work

### 1. llama.cpp Training Implementation ⚠️

**File**: `flutter_app/ios/Runner/TrainingController.swift`

The `executeTraining()` method is currently a placeholder. It needs:

1. **Load base model**:

   ```swift
   let model = llama_load_model_from_file(modelPath, ...)
   ```

2. **Load blank LoRA**:

   ```swift
   let blankLora = llama_load_lora_from_file(blankLoraPath, ...)
   ```

3. **Initialize optimizer**:

   ```swift
   let opt = llama_opt_init(
     model: model,
     lora: blankLora,
     trainingDataPath: trainingDataPath,
     ...
   )
   ```

4. **Run training epochs**:

   ```swift
   for step in 0..<stepCount {
     let loss = llama_opt_epoch(opt, step: step)
     lossTrend.append(loss)
   }
   ```

5. **Save trained adapter**:

   ```swift
   llama_save_lora(opt.trainedLora, userLoraPath)
   ```

6. **Convert to GGUF** (if needed):
   - Use `convert_lora_to_gguf.py` or equivalent Swift code

### 2. Blank LoRA Creation ⚠️

**File**: `flutter_app/ios/Runner/TrainingController.swift`

The `createBlankLoraIfNeeded()` method needs to:

1. Create a blank LoRA adapter with rank 8
2. Save to `blank_lora.gguf`
3. Use llama.cpp APIs to create blank adapter

### 3. GGUF Conversion ⚠️

After training, the adapter needs to be converted to GGUF format. Options:

1. **Use Python script** (via subprocess):

   ```swift
   let process = Process()
   process.executableURL = URL(fileURLWithPath: "/usr/bin/python3")
   process.arguments = [
     "convert_lora_to_gguf.py",
     trainedLoraPath,
     "--base", baseModelPath,
     "--outfile", userLoraPath,
   ]
   ```

2. **Use llama.cpp C++ APIs directly** (preferred):
   - Call `llama_convert_lora_to_gguf()` if available
   - Or implement conversion in Swift

### 4. Android Implementation ⚠️

The Android side needs:

1. **TrainingController.kt**: Mirror of Swift implementation
2. **Method channel registration**: In MainActivity.kt
3. **llama.cpp NDK integration**: For training on Android

### 5. Spec Update ⚠️

**File**: `specs/034-on-device-lora/data-model.md`

Update to reflect set-level training format instead of workout-level `WorkoutLogSummary`.

## Testing

### Unit Tests

```dart
// Test training data transformer
test('buildSetLevelSamples joins music logs to sets', () async {
  // ... test implementation
});

// Test JSONL formatting
test('formatAsJSONL produces valid JSONL', () {
  // ... test implementation
});
```

### Integration Tests

1. **End-to-end training flow**:
   - Create mock workout logs
   - Create mock music logs
   - Run transformer
   - Verify JSONL output
   - Call native training (mock)

2. **Background task scheduling**:
   - Schedule BG task
   - Verify registration
   - Test expiration handling

## Success Criteria

✅ Set-level training data with music correlation
✅ Training data transformer joins all data sources
✅ JSONL formatter produces valid training data
✅ Flutter service orchestrates pipeline
✅ Native training controller validates constraints
✅ Background tasks scheduled
✅ Method channels wired up

⚠️ llama.cpp training implementation (placeholder)
⚠️ GGUF conversion (needs implementation)
⚠️ Blank LoRA creation (needs implementation)
⚠️ Android parity (not started)

## Next Steps

1. **Implement llama.cpp training** in `TrainingController.swift`
2. **Implement blank LoRA creation**
3. **Implement GGUF conversion**
4. **Add Android implementation**
5. **Update spec** to reflect set-level format
6. **Add unit tests**
7. **Test end-to-end** with real data

## Notes

- Training data is **set-level** to preserve track → performance correlation
- Music logs are matched to sets by `exerciseId` + `setIndex`
- Taste preferences (starred/skipped) are included per track
- Training runs only when device is charging and battery > 20%
- Background tasks require Wi-Fi and charging

## Related

^[source-materials/mirrors/doctrine/USER_LORA_TRAINING_IMPLEMENTATION.md]
