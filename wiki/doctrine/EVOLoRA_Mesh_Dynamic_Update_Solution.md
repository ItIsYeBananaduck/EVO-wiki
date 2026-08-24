---
title: EVOLoRA_Mesh_Dynamic_Update_Solution
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVOLoRA_Mesh_Dynamic_Update_Solution.md
updated: 2026-07-24
---

# EVOLoRA Mesh Dynamic LoRA Update Solution

**Problem**: Need to update LoRA adapters **nightly on-device**, but llama.cpp requires GGUF format while training produces safetensors.

**Constraint**: Can't "freeze" adapters (pre-convert and store) - must support dynamic updates.

---

## Architecture Options

### Option 1: Server-Side Conversion + Nightly Download (Recommended)

**Flow**:

```
Nightly:
1. Server: Federated learning aggregates user data → Train LoRA (safetensors)
2. Server: Convert safetensors → GGUF (fast, has compute)
3. Server: Upload GGUF to R2
4. Device: Download GGUF adapters (nightly sync)
5. Device: Load GGUF in llama.cpp
```

**Pros**:

- ✅ Conversion happens server-side (fast, no device overhead)
- ✅ Device just downloads pre-converted GGUF
- ✅ Still "nightly updates" (device gets fresh adapters daily)
- ✅ Works with existing llama.cpp integration

**Cons**:

- ⚠️ Requires server-side conversion pipeline
- ⚠️ Network dependency for updates

**Implementation**:

- Add conversion step to federated learning aggregation pipeline
- Upload GGUF to R2 (not safetensors)
- Device downloads GGUF via `AliceAssetDownloadManager`

---

### Option 2: On-Device Conversion (Heavy)

**Flow**:

```
Nightly:
1. Device: Download safetensors from R2
2. Device: Convert safetensors → GGUF (on-device)
3. Device: Load GGUF in llama.cpp
```

**Pros**:

- ✅ No server-side conversion needed
- ✅ Device has full control

**Cons**:

- ❌ **Too slow/heavy** for mobile devices
- ❌ Conversion requires Python + transformers + torch
- ❌ Battery drain, storage overhead
- ❌ Not practical for nightly updates

**Verdict**: ❌ **Not viable** - too resource-intensive

---

### Option 3: Hybrid - Cache Converted, Update Incrementally

**Flow**:

```
Initial:
1. Server: Convert all adapters to GGUF, upload to R2
2. Device: Download and cache GGUF adapters

Nightly:
1. Server: Train new LoRA (safetensors)
2. Server: Convert to GGUF
3. Server: Upload delta/changes only
4. Device: Download delta, merge with cached GGUF
5. Device: Load updated GGUF
```

**Pros**:

- ✅ Smaller downloads (deltas only)
- ✅ Faster updates

**Cons**:

- ❌ Complex delta merging logic
- ❌ GGUF doesn't support incremental updates natively
- ❌ Risk of corruption if merge fails

**Verdict**: ⚠️ **Complex** - may not be worth it

---

### Option 4: Alternative Inference Engine (Major Change)

**Approach**: Use inference engine that supports safetensors natively

**Options**:

- **MLX** (Apple Silicon): Supports safetensors, but iOS deployment complex
- **CoreML**: Convert to CoreML format, but LoRA support unclear
- **ONNX Runtime**: Limited LoRA support
- **Transformers.js**: Web-based, not native

**Pros**:

- ✅ Native safetensors support
- ✅ No conversion needed

**Cons**:

- ❌ **Major architecture change** (lose llama.cpp optimizations)
- ❌ Performance may be worse
- ❌ iOS deployment complexity
- ❌ May not support dynamic LoRA loading

**Verdict**: ❌ **Too disruptive** - not worth losing llama.cpp benefits

---

## Recommended Solution: Option 1 (Server-Side Conversion)

### Implementation Plan

#### Step 1: Add Conversion to Federated Learning Pipeline

**File**: `federated-server/src/aggregate_and_train.py` (or similar)

```python
def aggregate_and_train_lora():
    """Aggregate federated deltas and train LoRA adapter."""
    # 1. Aggregate deltas from users
    aggregated_delta = aggregate_federated_deltas()

    # 2. Train LoRA adapter (safetensors)
    lora_dir = train_lora_adapter(aggregated_delta)
    # Output: lora_dir/adapter_model.safetensors

    # 3. Convert to GGUF
    gguf_path = convert_lora_to_gguf(
        lora_dir=lora_dir,
        base_model="microsoft/Phi-3-mini-4k-instruct",
        output_path=f"{lora_dir}.gguf"
    )

    # 4. Upload GGUF to R2
    upload_to_r2(gguf_path, f"alice-assets/adapters/global/user/global_user_lora.gguf")

    return gguf_path
```

#### Step 2: Update Device Download Logic

**File**: `flutter_app/lib/features/alice/domain/alice_asset_download_manager.dart`

```dart
// Add LoRA adapters to download list
static const List<_AliceAsset> _assets = <_AliceAsset>[
  // ... existing assets ...
  _AliceAsset(
    name: 'User LoRA Adapter (GGUF)',
    storagePath: 'adapters/user/user_lora.gguf',
    relativeTarget: 'AliceAssets/adapters/user/user_lora.gguf',
    useStreaming: true,
    expectedSizeBytes: 50 * 1024 * 1024, // ~50MB estimate
  ),
  _AliceAsset(
    name: 'Global User LoRA Adapter (GGUF)',
    storagePath: 'adapters/global/user/global_user_lora.gguf',
    relativeTarget: 'AliceAssets/adapters/global/user/global_user_lora.gguf',
    useStreaming: true,
    expectedSizeBytes: 100 * 1024 * 1024, // ~100MB estimate
  ),
  // ... etc for T, GT adapters ...
];
```

#### Step 3: Nightly Update Trigger

**File**: `flutter_app/lib/features/alice/domain/alice_asset_download_manager.dart`

```dart
/// Check for adapter updates nightly
Future<void> checkForAdapterUpdates() async {
  // Get last update timestamp
  final lastUpdate = await _getLastAdapterUpdateTime();
  final now = DateTime.now();

  // Check if 24 hours have passed
  if (now.difference(lastUpdate).inHours < 24) {
    return; // Not time for update yet
  }

  // Download updated adapters
  await downloadAdapters([
    'adapters/user/user_lora.gguf',
    'adapters/global/user/global_user_lora.gguf',
    // ... etc
  ]);

  // Update timestamp
  await _saveLastAdapterUpdateTime(now);
}
```

#### Step 4: Background Task Integration

**File**: `flutter_app/lib/features/alice/domain/alice_asset_download_manager.dart`

```dart
/// Register nightly adapter update task
void registerNightlyAdapterUpdate() {
  // Use existing background task infrastructure
  NativeSchedulerBridge.scheduleTask(
    identifier: 'com.evo.evotraining.nightly_adapter_update',
    earliestBeginDate: _getNextMidnight(),
    requiresNetworkConnectivity: true,
    callback: () async {
      await checkForAdapterUpdates();
    },
  );
}
```

---

## Update Frequency Strategy

### Per-User Adapter (U)

- **Update**: Weekly (less frequent, user patterns change slowly)
- **Trigger**: After weekly federated aggregation
- **Size**: ~50MB per user

### Per-Trainer Adapter (T)

- **Update**: Weekly (trainer patterns change slowly)
- **Trigger**: After trainer action aggregation
- **Size**: ~50MB per trainer

### Global User Adapter (GU)

- **Update**: **Nightly** (aggregated across all users)
- **Trigger**: After nightly federated learning aggregation
- **Size**: ~100MB (larger, more training data)

### Global Trainer Adapter (GT)

- **Update**: **Nightly** (aggregated across all trainers)
- **Trigger**: After nightly trainer pattern aggregation
- **Size**: ~100MB

---

## Conversion Performance

**Server-Side Conversion** (recommended):

- **Time**: ~10-30 seconds per adapter (depends on size)
- **Resources**: Server has CPU/GPU, can parallelize
- **Cost**: Minimal (runs during aggregation)

**On-Device Conversion** (not recommended):

- **Time**: ~5-10 minutes per adapter (mobile CPU)
- **Resources**: High battery drain, storage overhead
- **User Impact**: App unusable during conversion

---

## Implementation Checklist

- [ ] Add `convert_lora_to_gguf.py` to federated server
- [ ] Integrate conversion into aggregation pipeline
- [ ] Update R2 upload to use GGUF format
- [ ] Add adapter assets to `AliceAssetDownloadManager`
- [ ] Implement nightly update check
- [ ] Register background task for adapter updates
- [ ] Add adapter version tracking (to detect updates)
- [ ] Test conversion pipeline end-to-end
- [ ] Test device download and loading

---

## Alternative: Incremental Updates

If full adapter replacement is too slow, consider:

1. **Version-based updates**: Only download if version changed
2. **Delta compression**: Compress GGUF files (they compress well)
3. **Lazy loading**: Download adapters on-demand, not all at once
4. **Caching**: Keep old adapters until new ones are verified

---

## Summary

**Solution**: Server-side conversion + nightly device download

- ✅ Conversion happens server-side (fast, efficient)
- ✅ Device downloads pre-converted GGUF (small, fast)
- ✅ Still "nightly updates" (device gets fresh adapters daily)
- ✅ Works with existing llama.cpp integration
- ✅ No architecture changes needed
- ✅ Integrates with existing `backgroundModelUpdater.ts` infrastructure

**Key Insight**: "Nightly updates" doesn't mean "convert on-device" - it means "device gets fresh adapters nightly". The conversion happens server-side during federated aggregation, then device downloads the GGUF files via existing nightly background tasks.

**Integration Points**:

- Use existing `backgroundModelUpdater.ts` for nightly checks
- Use existing `modelUpdater.ts` download infrastructure
- Add conversion step to federated server aggregation pipeline
- Store GGUF adapters in R2 (same as base model)

## Related

^[source-materials/mirrors/doctrine/EVOLoRA_Mesh_Dynamic_Update_Solution.md]
