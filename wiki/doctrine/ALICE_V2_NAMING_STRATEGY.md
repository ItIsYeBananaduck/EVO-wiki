---
title: ALICE_V2_NAMING_STRATEGY
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/ALICE_V2_NAMING_STRATEGY.md
updated: 2026-07-24
---

# Alice v2 Naming Strategy for Seamless Integration

## Alice v2 Overview

**Alice v2** is a foundational retraining on three comprehensive datasets:

1. **Fitness** - Comprehensive fitness knowledge and coaching
2. **Nutrition** - Nutrition science and dietary guidance
3. **Mental Health & Teaching** - Mental wellness and educational approaches

These datasets form the **backbone** of Alice's knowledge across the entire suite, providing deep domain expertise in all core areas of the application.

**Key Difference from v1**: v2 is not just an incremental update, but a complete retraining on massive, domain-specific datasets that will serve as the foundation for all Alice interactions.

## Current File Structure

### Base Model

- **Storage Path**: `models/alice-phi3-q4.gguf`
- **Local Path**: `AliceAssets/models/alice-phi3-q4.gguf`
- **Referenced in**:
  - `alice_asset_download_manager.dart` (line 109-114)
  - `LlamaEngine.swift` (lines 92, 110, 445, 473, 487)

### ENF Adapter

- **Storage Path**: `adapters/enforcer/enforcer_lora.gguf`
- **Local Path**: `AliceAssets/adapters/enforcer/enforcer_lora.gguf`
- **Referenced in**:
  - `alice_asset_download_manager.dart` (line 121-126)
  - `LlamaEngine.swift` `getEnforcerAdapterPath()` (line 513, 533, 546)

### VOICE Adapter

- **Storage Path**: `adapters/voice/voice_lora.gguf`
- **Local Path**: `AliceAssets/adapters/voice/voice_lora.gguf`
- **Referenced in**:
  - `alice_asset_download_manager.dart` (line 133-138)
  - `LlamaEngine.swift` `getVoiceAdapterPath()` (line 570, 590, 603)

## Option 1: Replace In Place (Recommended for Production)

**Strategy**: Use same filenames, update file sizes, users automatically re-download v2 when app detects size mismatch.

### How It Works

The app **automatically detects v2** by checking file sizes:

1. **On app launch**, `_ensureAsset()` checks if file exists
2. **For GGUF files**, it compares `existingSize` vs `expectedSizeBytes`
3. **If sizes don't match** (v1 vs v2), it:
   - Deletes the old v1 file
   - Re-downloads v2 from R2 (download proceeds regardless of server size)
   - Verifies v2 size matches after download

**Code Flow** (from `alice_asset_download_manager.dart`):

**Step 1: Detect mismatch and delete** (lines 829-840):

```dart
// For GGUF files, require exact match; delete if different
final bool isGguf = asset.relativeTarget.toLowerCase().endsWith('.gguf');
if (isGguf) {
  if (existingSize != asset.expectedSizeBytes!) {
    print('AliceAssets: [_ensureAsset] ${asset.name} size mismatch (GGUF), deleting and re-downloading');
    await store.deleteFileIfExists(asset.relativeTarget);
    deletedExisting = true;  // Marks that we deleted old file
  }
}
```

**Step 2: Download proceeds** (lines 884-896):

```dart
// Download happens regardless - deletedExisting flag ensures fresh download
if (asset.useStreaming) {
  final int resumeOffset = (!deletedExisting && ...) ? existingSize : 0;
  await _downloadAssetStreamedToSharedStore(...);  // Downloads v2
}
```

**Step 3: Post-download verification** (lines 670-678):

```dart
// After download, verifies final size matches expectedSizeBytes
if (asset.expectedSizeBytes != null && finalSize != asset.expectedSizeBytes!) {
  final int diff = (finalSize - asset.expectedSizeBytes!).abs();
  final int tolerance = (asset.expectedSizeBytes! * 0.01).round().clamp(1, 1024 * 1024);
  if (diff > tolerance) {
    throw StateError('File size mismatch!');  // Fails if size doesn't match
  }
}
```

**Important**: The download **WILL proceed** even if server file size doesn't match `expectedSizeBytes`. The verification happens **after** download completes. If verification fails, the download throws an error, but the file is already downloaded.

**For v2 rollout**: Make sure R2 file size matches the `expectedSizeBytes` you set in code, or the post-download verification will fail.

### Naming

- Base model: `alice-phi3-q4.gguf` (same name)
- ENF adapter: `enforcer_lora.gguf` (same name)
- VOICE adapter: `voice_lora.gguf` (same name)

### Steps

1. **Get v2 file sizes** (before uploading):

   ```bash
   # Get actual file sizes from your v2 files
   ls -lh training/enf_lora/output/gguf/alice-phi3-v2-q4.gguf
   ls -lh training/enf_lora/output/gguf/enforcer_lora_v2.gguf
   ls -lh training/enf_lora/output/gguf/voice_lora_v2.gguf

   # Or use stat to get exact bytes
   stat -f%z training/enf_lora/output/gguf/alice-phi3-v2-q4.gguf  # macOS
   # or
   stat -c%s training/enf_lora/output/gguf/alice-phi3-v2-q4.gguf  # Linux
   ```

2. **Upload v2 to R2** (replace existing files):

   ```bash
   # Base model
   wrangler r2 object put alice-assets/models/alice-phi3-q4.gguf \
     --file=./training/enf_lora/output/gguf/alice-phi3-v2-q4.gguf

   # ENF adapter
   wrangler r2 object put alice-assets/adapters/enforcer/enforcer_lora.gguf \
     --file=./training/enf_lora/output/gguf/enforcer_lora_v2.gguf

   # VOICE adapter
   wrangler r2 object put alice-assets/adapters/voice/voice_lora.gguf \
     --file=./training/enf_lora/output/gguf/voice_lora_v2.gguf
   ```

3. **Update File Sizes** in `alice_asset_download_manager.dart` (use sizes from step 1):

   ```dart
   _AliceAsset(
     name: 'Alice Phi-3 (GGUF)',
     storagePath: 'models/alice-phi3-q4.gguf',
     relativeTarget: 'AliceAssets/models/alice-phi3-q4.gguf',
     useStreaming: true,
     expectedSizeBytes: <V2_SIZE>, // Update with v2 size (different from v1)
   ),
   _AliceAsset(
     name: 'ENF LoRA Adapter',
     storagePath: 'adapters/enforcer/enforcer_lora.gguf',
     relativeTarget: 'AliceAssets/adapters/enforcer/enforcer_lora.gguf',
     useStreaming: true,
     expectedSizeBytes: <V2_SIZE>, // Update with v2 size
   ),
   _AliceAsset(
     name: 'VOICE LoRA Adapter',
     storagePath: 'adapters/voice/voice_lora.gguf',
     relativeTarget: 'AliceAssets/adapters/voice/voice_lora.gguf',
     useStreaming: true,
     expectedSizeBytes: <V2_SIZE>, // Update with v2 size
   ),
   ```

4. **Deploy app update** with new `expectedSizeBytes` values

5. **Automatic rollout**:
   - Users with v1: App detects size mismatch → deletes v1 → downloads v2
   - New users: Download v2 directly
   - All seamless, no manual intervention needed

**Pros**:

- Zero code changes (just update sizes)
- Automatic rollout (users get v2 on next app launch)
- No versioning complexity
- Size-based detection ensures correct version

**Cons**:

- All users must re-download (but app handles this automatically)
- Can't A/B test v1 vs v2
- Requires app update to change `expectedSizeBytes`

---

## Option 2: Versioned Names (For A/B Testing)

**Strategy**: Use `v2` suffix, add code to support both versions.

### Naming

- Base model: `alice-phi3-v2-q4.gguf`
- ENF adapter: `enforcer_lora_v2.gguf`
- VOICE adapter: `voice_lora_v2.gguf`

### Steps

1. **Upload to R2** (new files alongside v1):

   ```bash
   # Base model
   wrangler r2 object put alice-assets/models/alice-phi3-v2-q4.gguf \
     --file=./training/enf_lora/output/gguf/alice-phi3-v2-q4.gguf

   # ENF adapter
   wrangler r2 object put alice-assets/adapters/enforcer/enforcer_lora_v2.gguf \
     --file=./training/enf_lora/output/gguf/enforcer_lora_v2.gguf

   # VOICE adapter
   wrangler r2 object put alice-assets/adapters/voice/voice_lora_v2.gguf \
     --file=./training/enf_lora/output/gguf/voice_lora_v2.gguf
   ```

2. **Add v2 Assets** to `alice_asset_download_manager.dart`:

   ```dart
   // Add after existing assets
   baseAssets.add(
     _AliceAsset(
       name: 'Alice Phi-3 v2 (GGUF)',
       storagePath: 'models/alice-phi3-v2-q4.gguf',
       relativeTarget: 'AliceAssets/models/alice-phi3-v2-q4.gguf',
       useStreaming: true,
       expectedSizeBytes: <V2_SIZE>,
     ),
   );
   baseAssets.add(
     _AliceAsset(
       name: 'ENF LoRA Adapter v2',
       storagePath: 'adapters/enforcer/enforcer_lora_v2.gguf',
       relativeTarget: 'AliceAssets/adapters/enforcer/enforcer_lora_v2.gguf',
       useStreaming: true,
       expectedSizeBytes: <V2_SIZE>,
     ),
   );
   baseAssets.add(
     _AliceAsset(
       name: 'VOICE LoRA Adapter v2',
       storagePath: 'adapters/voice/voice_lora_v2.gguf',
       relativeTarget: 'AliceAssets/adapters/voice/voice_lora_v2.gguf',
       useStreaming: true,
       expectedSizeBytes: <V2_SIZE>,
     ),
   );
   ```

3. **Update Native Code** to support version selection:

   **iOS** (`LlamaEngine.swift`):

   ```swift
   // Add version parameter or feature flag
   private let useV2Model = true // or read from config

   @objc func getModelPath() -> String? {
       let modelName = useV2Model ? "alice-phi3-v2-q4.gguf" : "alice-phi3-q4.gguf"
       // ... existing path logic with modelName
   }

   @objc func getEnforcerAdapterPath() -> String? {
       let adapterName = useV2Model ? "enforcer_lora_v2.gguf" : "enforcer_lora.gguf"
       // ... existing path logic with adapterName
   }

   @objc func getVoiceAdapterPath() -> String? {
       let adapterName = useV2Model ? "voice_lora_v2.gguf" : "voice_lora.gguf"
       // ... existing path logic with adapterName
   }
   ```

   **Android** (`LlamaPlugin.kt`):

   ```kotlin
   // Similar changes to adapter path methods
   ```

**Pros**:

- Can A/B test v1 vs v2
- Can rollback easily
- Gradual rollout possible

**Cons**:

- Requires code changes
- More complex deployment
- Users download both versions (more storage)

---

## Recommendation

**For Production**: Use **Option 1** (Replace In Place)

- Simplest deployment
- Automatic rollout
- No code changes needed
- Users get v2 seamlessly on next app launch

**For Testing**: Use **Option 2** (Versioned Names)

- If you need to A/B test v1 vs v2
- If you want gradual rollout
- If you need easy rollback

---

## Complete Workflow Summary

### Step-by-Step Process

1. **Prepare v2 files** (convert to GGUF if needed)
2. **Get file sizes** (before uploading):

   ```bash
   # Get exact byte sizes
   stat -f%z training/enf_lora/output/gguf/alice-phi3-v2-q4.gguf
   stat -f%z training/enf_lora/output/gguf/enforcer_lora_v2.gguf
   stat -f%z training/enf_lora/output/gguf/voice_lora_v2.gguf
   ```

3. **Upload v2 to R2** (same filenames, replaces v1):

   ```bash
   wrangler r2 object put alice-assets/models/alice-phi3-q4.gguf --file=./v2-file.gguf
   wrangler r2 object put alice-assets/adapters/enforcer/enforcer_lora.gguf --file=./v2-file.gguf
   wrangler r2 object put alice-assets/adapters/voice/voice_lora.gguf --file=./v2-file.gguf
   ```

4. **Update code** with v2 file sizes in `alice_asset_download_manager.dart`:

   ```dart
   expectedSizeBytes: <V2_BASE_MODEL_SIZE>,  // From step 2
   expectedSizeBytes: <V2_ENF_SIZE>,          // From step 2
   expectedSizeBytes: <V2_VOICE_SIZE>,        // From step 2
   ```

5. **Deploy app update** (users will auto-update on next launch)

### Why This Order?

- **Upload first**: R2 has v2 ready
- **Update code second**: App knows v2 sizes, detects v1 mismatch, downloads v2
- **Deploy**: Users get v2 seamlessly

**Critical**: R2 file sizes must match `expectedSizeBytes` (or be within 1% tolerance) or post-download verification will fail.

---

## Testing Checklist

- [ ] Upload v2 files to R2
- [ ] Update file sizes in `alice_asset_download_manager.dart`
- [ ] Test download on iOS simulator
- [ ] Test download on Android emulator
- [ ] Test model loading in app
- [ ] Test inference with v2 model
- [ ] Verify continuation logic works with v2
- [ ] Test on physical device
- [ ] Verify fitness domain knowledge (v2 dataset)
- [ ] Verify nutrition domain knowledge (v2 dataset)
- [ ] Verify mental health/teaching knowledge (v2 dataset)
- [ ] Test cross-domain capabilities (fitness + nutrition + mental health)

## Notes on v2 Training

**Training Datasets**:

- **Fitness**: Comprehensive fitness knowledge and coaching methodologies
- **Nutrition**: Nutrition science, dietary guidance, and meal planning
- **Mental Health & Teaching**: Mental wellness strategies and educational approaches

**Expected Improvements**:

- Deeper domain expertise across all three areas
- Better cross-domain integration (e.g., fitness + nutrition + mental health)
- More comprehensive answers leveraging all three knowledge bases
- Foundation for future domain-specific features

**Integration Considerations**:

- v2 will be the backbone for all Alice interactions
- Continuation logic should work seamlessly with v2's potentially longer, more comprehensive answers
- Capability map queries should reflect v2's expanded knowledge base

## Related

^[source-materials/mirrors/doctrine/ALICE_V2_NAMING_STRATEGY.md]
