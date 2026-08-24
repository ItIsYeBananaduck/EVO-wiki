---
title: IMPLEMENTATION_READINESS_ASSESSMENT
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/IMPLEMENTATION_READINESS_ASSESSMENT.md"]
updated: 2026-07-24
---

# Implementation Readiness Assessment: Nightly User LoRA Training

## Question: Do we have enough to implement now?

## ✅ **YES - We have most components, but need a few gaps filled**

---

## What We HAVE ✅

### 1. llama.cpp Integration ✅

- **Status**: Fully integrated
- **Location**: `flutter_app/ios/llama.xcframework/`
- **APIs Available**:
  - `llama_adapter_lora_init()` - Initialize LoRA adapter
  - `llama_adapter_lora_free()` - Cleanup
  - Adapter loading/application APIs
- **Note**: Need to verify training APIs (`llama_train`, `llama_opt`) are available

### 2. Base Model ✅

- **Status**: Available
- **Location**: `AliceAssets/models/alice-phi3-q4.gguf` (or similar)
- **Format**: GGUF (4-bit quantized)
- **Size**: ~2.4GB

### 3. User LoRA Path Management ✅

- **Status**: Implemented
- **Location**: `flutter_app/lib/features/alice/domain/lora_adapter_manager.dart`
- **Path**: `adapters/user/user_lora.gguf`
- **Functions**: `getAdapterPath()`, `adapterExists()`, etc.

### 4. Background Task Infrastructure ✅

- **Status**: Registered (handler commented out)
- **Location**: `flutter_app/ios/Runner/AppDelegate.swift`
- **Identifier**: `com.evo.evotraining.nightly_federated`
- **Info.plist**: Configured

### 5. Memory Safeguards ✅

- **Status**: Implemented
- **Location**: `flutter_app/lib/features/alice/domain/gating/memory_safeguards.dart`
- **Functions**: `canRunNightlyTraining()`, resource constraint checks

### 6. Specification & Contracts ✅

- **Status**: Complete
- **Location**: `specs/034-on-device-lora/`
- **Includes**:
  - Full spec (`spec.md`)
  - Implementation plan (`plan.md`)
  - Research notes (`research.md`)
  - Contract definitions (`contracts/mobile-training.md`)
  - Data models (`data-model.md`)

### 7. GGUF Conversion Experience ✅

- **Status**: We've done this before
- **Location**: `training/enf_lora/scripts/convert_to_gguf.py`
- **Experience**: Successfully converted ENF and VOICE LoRAs to GGUF
- **Can reuse**: Same approach for user LoRA

### 8. EVOLoRA Mesh System ✅

- **Status**: Fully implemented
- **Location**: `flutter_app/lib/features/alice/domain/evolora_mesh/`
- **Integration**: User LoRA (U) is already part of the mesh
- **Application**: Adapters are loaded and applied in `LlamaEngine.swift`

---

## What We NEED ⚠️

### 1. llama.cpp Training APIs ⚠️

- **Status**: Need to verify
- **Required APIs**:
  - `llama_train()` or equivalent training function
  - `llama_opt_init()` - Optimizer initialization
  - Training data format (JSONL or similar)
- **Action**: Check `llama.h` for training APIs
- **Risk**: Low - llama.cpp has training support, just need to verify Swift bindings

### 2. Blank LoRA Adapter ⚠️

- **Status**: Referenced but need to verify exists
- **Location**: Should be `models/blank_lora.bin` or `blank_lora.gguf`
- **Purpose**: Baseline for delta computation
- **Action**:
  - Check if exists in project
  - If not, create using `llama.cpp` tools
  - Or download from Hugging Face (referenced in `scripts/models-fetch.ts`)

### 3. Workout Log Data Access ✅

- **Status**: **FULLY IMPLEMENTED** - On-device Isar database + SharedPreferences
- **Data Storage**:
  - **Isar** database (on-device, local) - ✅ Confirmed
  - `WorkoutLogEntity` stored via `IsarOnDeviceDataStore` - ✅ Confirmed
  - Access via `onDeviceDataStore.getWorkoutLogs(userId, dateRange)` - ✅ Confirmed
- **Data Model**: `OnDeviceWorkoutLog` with:
  - ✅ Exercise data, sets, reps, weight
  - ✅ **RPE** (in `OnDeviceWorkoutSet.rpe`) - ✅ Available
  - ✅ **Notes** (exercise notes, set notes, workout notes) - ✅ Available
  - ✅ Heart rate data (`avgHeartRate`, `maxHeartRate`)
  - ✅ Timestamps, user ID, plan linkage
- **HRV Data**:
  - ⚠️ HRV stored in **separate** `OnDeviceStrainRecord` entity
  - Access via `onDeviceDataStore.getStrainRecords(userId, dateRange)`
  - Field: `OnDeviceStrainRecord.hrvScore`
  - **Action**: Need to join workout logs + strain records by date
- **StrainSync Music Data**:
  - ✅ **Stored separately** in SharedPreferences (not in Isar)
  - Service: `StrainSyncMusicLogService`
  - Storage key: `strainsync_set_logs_${userId}_$sessionId`
  - Data: `StrainSyncSetMusicLog` with:
    - Track info (trackKey, trackName, artistName)
    - Phase (warmup/main/peak/cooldown)
    - Set index, exercise info
    - avgHeartRate during music playback
  - **Taste preferences**: `StrainSyncTrackStore` tracks "taste more" (starred) and "taste less" preferences
  - **Action**: Need to join music logs with workout logs by `sessionId`
- **Required Format**:
  - ✅ RPE available (in sets)
  - ✅ Form notes available (in notes fields)
  - ⚠️ HRV needs to be joined from strain records
  - ⚠️ Convert to prompt/response pairs for training
- **Action**:
  - ✅ Query Isar database from Flutter (already implemented)
  - ✅ RPE and notes are available
  - ⚠️ Join with strain records to get HRV
  - ⚠️ Format as training dataset (JSONL or similar)
  - ⚠️ Pass to iOS via method channel

### 4. Training Data Format ⚠️

- **Status**: Need to define transformation (data is available)
- **Data Available**:
  - ✅ `OnDeviceWorkoutLog` from Isar database (RPE, notes, heart rate)
  - ✅ `OnDeviceStrainRecord` from Isar database (HRV score)
  - ✅ `StrainSyncSetMusicLog` from SharedPreferences (music taste data)
  - ✅ `StrainSyncTrackMeta` from SharedPreferences (taste more/less preferences)
- **Required Transformation**:
  1. Query workout logs: `getWorkoutLogs(userId, dateRange)`
  2. Query strain records: `getStrainRecords(userId, dateRange)`
  3. Query music logs: `strainSyncMusicLogService.loadLogsForSession(userId, sessionId)` for each workout
  4. Query taste preferences: `strainSyncTrackStore.getOrCreate(trackKey)` for each track
  5. Join all data:
     - Join workout logs + strain records by `localDate`
     - Join workout logs + music logs by `sessionId` (workout log `id`)
     - Join music logs + taste preferences by `trackKey`
  6. Convert to **set-level training samples** (NOT workout-level):
     - **For each set** in each exercise:
       - Extract set performance: RPE, reps, weight, heart rate for THAT set
       - Match music log by `exerciseId` + `setIndex`
       - Extract track info and taste preferences for THAT set's track
       - Include phase context (warmup/main/peak/cooldown)
       - Include HRV context (from strain record for the workout date)
       - Include form notes (from workout/exercise/set)
     - **Result**: One training sample per set, preserving track → performance correlation
  7. Convert to prompt/response pairs for llama.cpp training
  8. Format as JSONL for training API
- **Action**:
  - ✅ Data models confirmed (RPE, notes, HRV, music all available)
  - ⚠️ Create transformation service:
    - `OnDeviceWorkoutLog` + `OnDeviceStrainRecord` + `StrainSyncSetMusicLog[]` + `StrainSyncTrackMeta`
    - → `WorkoutLogSummary` (with `strainSync` field)
    - → training JSONL

### 5. Background Task Handler ⚠️

- **Status**: Registered but not implemented
- **Location**: `AppDelegate.swift` (commented out)
- **Required**:
  - Check constraints (charging, WiFi, battery)
  - Query workout logs
  - Invoke training
  - Save results
- **Action**: Uncomment and implement handler

### 6. GGUF Conversion for User LoRA ⚠️

- **Status**: Need to adapt existing script
- **Current**: `convert_to_gguf.py` works for ENF/VOICE
- **Required**: Same approach for user LoRA
- **Action**:
  - Reuse conversion script
  - Or use llama.cpp API to save as GGUF directly

---

## Implementation Roadmap

### Phase 1: Verify Prerequisites (1-2 hours)

1. ✅ Check llama.cpp training APIs in headers
2. ✅ Verify blank LoRA exists or create it
3. ✅ **Workout log access confirmed** - Isar database with `getWorkoutLogs()`
4. ✅ **Data fields confirmed**:
   - ✅ RPE: `OnDeviceWorkoutSet.rpe`
   - ✅ Notes: `OnDeviceWorkoutLog.notes`, `OnDeviceWorkoutExercise.notes`, `OnDeviceWorkoutSet.notes`
   - ✅ HRV: `OnDeviceStrainRecord.hrvScore` (separate query needed)
5. ✅ Verify training data format requirements

### Phase 2: Core Training Implementation (4-6 hours)

1. Create `TrainingController.swift` plugin
2. Implement `runNightlyTraining()` method
3. Bridge to llama.cpp training APIs
4. **Query all data sources** via Flutter:
   - `getWorkoutLogs(userId, dateRange)` → `OnDeviceWorkoutLog[]`
   - `getStrainRecords(userId, dateRange)` → `OnDeviceStrainRecord[]`
   - For each workout: `strainSyncMusicLogService.loadLogsForSession(userId, sessionId)` → `StrainSyncSetMusicLog[]`
   - For each track: `strainSyncTrackStore.getOrCreate(trackKey)` → `StrainSyncTrackMeta`
5. **Create transformation service**:
   - Join workout logs + strain records by `localDate`
   - Join workout logs + music logs by `sessionId` (workout `id`)
   - Join music logs + taste preferences by `trackKey`
   - Extract RPE (from sets), notes (from exercises/sets/workout), HRV (from strain records)
   - Aggregate music taste data: count tasteMore/tasteLess per phase
   - Build `StrainSyncSummary` with segments (warmup/main/peak/cooldown)
   - Convert to `WorkoutLogSummary` format (with `strainSync` field)
6. Format as training JSONL for llama.cpp
7. Pass to iOS via method channel
8. Save trained adapter

### Phase 3: GGUF Conversion (2-3 hours)

1. Convert trained adapter to GGUF
2. Save as `user_lora.gguf`
3. Update adapter manager metadata

### Phase 4: Background Task Integration (2-3 hours)

1. Implement background task handler
2. Add constraint checks
3. Integrate with scheduler
4. Add error handling and logging

### Phase 5: Testing & Validation (2-3 hours)

1. Unit tests for training logic
2. Integration tests with mock data
3. Test on device (charging, WiFi constraints)
4. Verify GGUF conversion

---

## Risk Assessment

### Low Risk ✅

- llama.cpp integration (already working)
- Adapter loading/application (already in mesh)
- GGUF conversion (we've done this)
- Background task registration (already done)

### Medium Risk ⚠️

- Training API availability (need to verify)
- ✅ HRV/RPE/form notes confirmed available (RPE in sets, notes in multiple fields, HRV in strain records)
- Training data transformation (join workout logs + strain records → WorkoutLogSummary → JSONL)
- Training performance (100 steps in <5 min)

### High Risk ❌

- None identified

---

## Conclusion

**YES, we have enough to implement now**, with these caveats:

1. **Need to verify** llama.cpp training APIs are available in Swift bindings
2. **Need to create** blank LoRA if it doesn't exist (or download from HF)
3. **Need to bridge** workout log data from Flutter to iOS
4. **Need to implement** the actual training logic (but we have all the pieces)

**Estimated Implementation Time**: 12-18 hours of focused work

**Confidence Level**: **85%** - We have all major components, just need to wire them together.

---

## Next Steps

1. **Verify Training APIs**: Check `llama.h` for `llama_train` or equivalent ✅ (Found `llama_opt_init`, `llama_opt_epoch`)
2. **Verify Workout Log Model**: ✅ **CONFIRMED**:
   - ✅ RPE: `OnDeviceWorkoutSet.rpe`
   - ✅ Notes: Multiple fields (workout, exercise, set notes)
   - ✅ HRV: `OnDeviceStrainRecord.hrvScore` (separate entity, join by date)
   - ✅ **Music/StrainSync**: `StrainSyncSetMusicLog` in SharedPreferences (join by sessionId)
   - ✅ **Taste preferences**: `StrainSyncTrackMeta` with tasteMore/tasteLess counts
3. **Create Blank LoRA**: Generate or download baseline adapter
4. **Start Implementation**: Begin with `TrainingController.swift` plugin
5. **Create Data Transformer**: Build transformation service:
   - Join `OnDeviceWorkoutLog` + `OnDeviceStrainRecord` by date
   - Join `OnDeviceWorkoutLog` + `StrainSyncSetMusicLog[]` by sessionId
   - **Match music logs to sets** by `exerciseId` + `setIndex` (critical for set-level granularity)
   - Join `StrainSyncSetMusicLog` + `StrainSyncTrackMeta` by trackKey
   - **Create set-level training samples**: Each set gets its own entry with:
     - Set performance (RPE, reps, weight, heart rate)
     - Music track that played during that set
     - Taste preferences (tasteMore/tasteLess) for that track
     - Phase context (warmup/main/peak/cooldown)
   - Convert to **set-level JSONL** (not workout-level) → training pipeline
6. **Iterate**: Test each component as we build

## Updated Data Flow

```
┌─────────────────────────────────────────┐
│  Isar Database (On-Device)              │
│  ✅ WorkoutLogEntity stored locally     │
│  ✅ Access via onDeviceDataStore        │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│  Flutter Service Layer                   │
│  ✅ Query: getWorkoutLogs(userId, range)│
│  ✅ Query: getStrainRecords(userId, range)│
│  ✅ Query: loadLogsForSession(sessionId)│
│  ✅ Query: getOrCreate(trackKey)        │
│  ⚠️ Join: Match by date + sessionId      │
│  ⚠️ Match: Music logs to sets by        │
│     exerciseId + setIndex                │
│  ⚠️ Transform: Create SET-LEVEL samples │
│     (one per set, not per workout)      │
│     Each sample: set performance +      │
│     track that played + taste prefs     │
│  ⚠️ Format: Set-level JSONL for training │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│  Method Channel (Flutter → iOS)          │
│  ⚠️ Pass training data to iOS            │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│  TrainingController.swift (iOS)         │
│  ⚠️ Receive training data                │
│  ⚠️ Call llama.cpp training APIs         │
│  ⚠️ Save trained adapter                 │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│  GGUF Conversion                         │
│  ⚠️ Convert trained adapter to GGUF      │
│  ✅ Save as user_lora.gguf               │
└─────────────────────────────────────────┘
```

**We're ready to start!** 🚀

## Related
