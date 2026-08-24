---
title: TRAINING_PIPELINE_INTEGRATION
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/TRAINING_PIPELINE_INTEGRATION.md
updated: 2026-07-24
---

# Training Pipeline Integration - Complete ✅

## Summary

The User LoRA training pipeline is now **fully integrated** into the app's nightly background worker. The service will automatically run at 3 AM every night alongside the federated sync.

## What Was Added

### 1. Nightly Worker Integration ✅

**File**: `flutter_app/lib/core/background/nightly_federated_worker.dart`

Added `_runUserLoRATraining()` function that:

- Gets user session from Supabase
- Initializes training service with Isar, music logs, and track store
- Schedules background tasks (idempotent)
- Runs training with last 7 days of data (100 steps)
- Logs results (doesn't fail entire nightly task if training fails)

### 2. Automatic Execution

The training now runs automatically:

- **When**: Every night at 3 AM (same time as federated sync)
- **Where**: Background worker (`nightly_federated_worker.dart`)
- **How**: Integrated into existing `callbackDispatcher()` function
- **Constraints**:
  - Requires user session (skips if not logged in)
  - Requires Isar initialized (skips if not available)
  - Runs independently (doesn't affect federated sync success)

## Pipeline Flow

```
1. App launches
   ↓
2. Workmanager initialized (main.dart)
   ↓
3. Nightly task registered for 3 AM
   ↓
4. Background worker wakes up at 3 AM
   ↓
5. Runs federated sync (existing)
   ↓
6. Runs User LoRA training (NEW)
   ├─ Get user session
   ├─ Initialize training service
   ├─ Schedule background tasks
   ├─ Prepare training data (set-level with music)
   ├─ Call native TrainingController
   └─ Log results
   ↓
7. Reschedule for next night
```

## What You Need to Do

### ✅ Nothing! It's Already Wired Up

The pipeline is complete and will run automatically. However, you still need to:

### 1. Implement llama.cpp Training (Required) ⚠️

**File**: `flutter_app/ios/Runner/TrainingController.swift`

The `executeTraining()` method is a placeholder. You need to:

```swift
private func executeTraining(trainingDataPath: String, stepCount: Int) -> TrainingResult {
    // 1. Load base model
    let model = llama_load_model_from_file(modelPath, ...)

    // 2. Load blank LoRA
    let blankLora = llama_load_lora_from_file(blankLoraPath, ...)

    // 3. Initialize optimizer
    let opt = llama_opt_init(
        model: model,
        lora: blankLora,
        trainingDataPath: trainingDataPath,
        ...
    )

    // 4. Run training epochs
    var lossTrend: [Double] = []
    for step in 0..<stepCount {
        let loss = llama_opt_epoch(opt, step: step)
        lossTrend.append(loss)
    }

    // 5. Save trained adapter
    llama_save_lora(opt.trainedLora, userLoraPath)

    // 6. Convert to GGUF (if needed)
    // ...

    return TrainingResult(...)
}
```

### 2. Implement Blank LoRA Creation (Required) ⚠️

**File**: `flutter_app/ios/Runner/TrainingController.swift`

The `createBlankLoraIfNeeded()` method needs to create a blank LoRA adapter:

```swift
private func createBlankLoraIfNeeded() -> Bool {
    // Use llama.cpp APIs to create blank LoRA (rank 8)
    // Save to blankLoraPath
    return true
}
```

### 3. Test the Pipeline (Recommended)

1. **Manual test**: Call the service directly from Flutter:

   ```dart
   final trainingService = UserLoRATrainingService(...);
   final report = await trainingService.runNightlyTraining(
     userId: user.id,
     daysBack: 7,
     stepCount: 100,
   );
   ```

2. **Background test**: Wait for 3 AM or trigger background task manually

3. **Check logs**: Look for `[nightly_federated_worker]` messages

## Current Status

✅ **Complete**:

- Set-level training data models
- Training data transformer (joins music + workout data)
- JSONL formatter
- Flutter training service
- Training controller plugin
- Native TrainingController (Swift)
- Background task scheduling
- Integration into nightly worker
- Spec updated to set-level format

⚠️ **Needs Implementation**:

- llama.cpp training calls (placeholder exists)
- Blank LoRA creation (placeholder exists)
- GGUF conversion (placeholder exists)
- Android parity (not started)

## Next Steps

1. **Implement llama.cpp training** in `TrainingController.swift.executeTraining()`
2. **Implement blank LoRA creation** in `TrainingController.swift.createBlankLoraIfNeeded()`
3. **Test end-to-end** with real workout data
4. **Add Android implementation** (mirror Swift code)

## Notes

- Training runs **automatically** at 3 AM every night
- It's **non-blocking** - won't fail the entire nightly task if training fails
- Requires **user session** - skips if user not logged in
- Requires **Isar** - skips if database not initialized
- Training data is **set-level** to preserve track → performance correlation

The pipeline is **ready to use** once you implement the llama.cpp training calls!

## Related

^[source-materials/mirrors/doctrine/TRAINING_PIPELINE_INTEGRATION.md]
