---
title: EVOLoRA_Mesh_Dual_Model_Format_Plan
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVOLoRA_Mesh_Dual_Model_Format_Plan.md
updated: 2026-07-24
---

# EVOLoRA Mesh Dual Model Format Plan

**Problem**: Different platforms need different model formats. Instead of nightly conversion, maintain two compatible models and download the appropriate one per platform.

**Solution**:

- **iOS**: MLX format (safetensors-based, native LoRA support)
- **Android**: llama.cpp GGUF format (already integrated, proven)

**Key**: Server converts adapters **ONCE** when they're updated (not nightly). Devices download pre-converted formats. **No nightly server calls needed!**

---

## Architecture Overview

### Model Storage (R2)

```
alice-assets/
├── models/
│   ├── alice-phi3-mlx-base-q4/          ← iOS (MLX format)
│   │   ├── model.safetensors
│   │   ├── config.json
│   │   └── tokenizer.json
│   └── alice-phi3-q4.gguf               ← Android (GGUF format, already have ✅)
├── adapters/
│   ├── ios/                              ← iOS adapters (safetensors)
│   │   ├── user_lora.safetensors
│   │   ├── {trainerId}_lora.safetensors
│   │   ├── global_user_lora.safetensors
│   │   └── global_trainer_lora.safetensors
│   └── android/                          ← Android adapters (GGUF)
│       ├── user_lora.gguf
│       ├── {trainerId}_lora.gguf
│       ├── global_user_lora.gguf
│       └── global_trainer_lora.gguf
```

### Platform Detection & Download

**On App Launch/Background Check**:

1. Check platform: `Platform.isIOS` or `Platform.isAndroid`
2. Download appropriate base model format (if not already downloaded)
3. Check for adapter updates (compare checksums/versions)
4. Download appropriate LoRA adapter formats if updated
5. **No nightly server calls**: Only checks when app launches or in background

---

## iOS: MLX Format

### Base Model

- **Format**: MLX (safetensors-based)
- **Size**: ~2GB (q4 quantized)
- **Conversion**: One-time (server-side or during training)
- **LoRA Support**: ✅ Native safetensors (no conversion)

### LoRA Adapters

- **Format**: Safetensors (from MLX training)
- **No conversion needed**: Direct from training pipeline
- **Dynamic loading**: ✅ Supported
- **Incremental training**: ✅ Supported

---

## Android: llama.cpp GGUF Format (Keep Current)

### Base Model

- **Format**: GGUF (already have `alice-phi3-q4.gguf`)
- **Size**: ~2GB (q4 quantized)
- **Conversion**: Already done ✅
- **LoRA Support**: ✅ GGUF LoRA adapters

### LoRA Adapters

- **Format**: GGUF (converted from safetensors, server-side)
- **Conversion**: One-time (server-side, safetensors → GGUF when adapter updates)
- **Dynamic loading**: ✅ Supported (llama.cpp supports multiple LoRAs)
- **Incremental training**: ❌ Not supported (but training happens in safetensors, then converted)

**Key**:

- **U/T adapters**: Converted **on-device** immediately after training (no server calls)
- **GU/GT adapters**: Converted **server-side** during weekly aggregation (only server-side operation)

---

## On-Device Conversion Workflow (U/T Adapters)

### User (U) & Trainer (T) Adapters - On-Device

**Training**: Daily (nightly on-device training)

**Process** (All On-Device, No Server Calls):

1. **Nightly (3 AM)**: On-device training completes → Produces `user_lora.safetensors`
2. **On-device conversion** (immediately after training):
   - **iOS**: Use safetensors directly (no conversion) ✅
   - **Android**: Convert safetensors → GGUF on-device (using llama.cpp's convert-lora-to-gguf)
3. **Store locally**:
   - iOS: `AliceAssets/adapters/user/user_lora.safetensors`
   - Android: `AliceAssets/adapters/user/user_lora.gguf`
4. **Upload delta to server** (for federated aggregation only, not for conversion):
   - Upload safetensors delta to server (for weekly GU/GT aggregation)
   - **No server conversion needed** - conversion stays on-device

**Key**: U/T adapters are converted **on-device** immediately after training. No server calls for conversion.

---

## Server-Side Conversion Workflow (GU/GT Adapters Only)

### Global User (GU) & Global Trainer (GT) Adapters - Server-Side

**Training**: Weekly (federated aggregation on server)

**Process** (Server-Side, Only for Weekly Aggregation):

1. **Weekly (Sunday night)**: Federated aggregation completes → Server produces `global_user_lora.safetensors`
2. **Server-side conversion** (immediately after aggregation):
   - **iOS**: Copy safetensors (no conversion) → Upload to R2
   - **Android**: Convert safetensors → GGUF → Upload to R2
3. **Upload to R2**:
   - Upload both formats to R2
   - Update manifest with checksums/versions
4. **Device downloads** (on app launch/background):
   - Compare local checksum with server checksum
   - Download if new version available
   - **Both platforms get weekly updates** ✅

**Key**: Only GU/GT adapters use server-side conversion (weekly aggregation). U/T adapters stay on-device.

### Conversion Script

**File**: `training/export_adapters_for_platforms.py` (CREATED)

```python
# Converts safetensors LoRA to platform-specific formats
python training/export_adapters_for_platforms.py \
    --safetensors training/alice-phi3-mlx/adapters/adapters.safetensors \
    --base-model models/alice-phi3-q4.gguf \
    --adapter-name user_lora \
    --output adapters/
```

This creates:

- `adapters/ios/user_lora.safetensors` (copy, no conversion)
- `adapters/android/user_lora.gguf` (converted from safetensors)

---

## Platform Detection & Download

### Flutter Download Manager

**File**: `flutter_app/lib/features/alice/domain/alice_asset_download_manager.dart` (MODIFY)

```dart
import 'dart:io' show Platform;

class AliceAssetDownloadManager {
  // Detect platform and download appropriate model
  Future<void> downloadBaseModel() async {
    if (Platform.isIOS) {
      // iOS: Download MLX format
      await downloadFromR2(
        remotePath: 'alice-assets/models/alice-phi3-mlx-base-q4/',
        localPath: 'AliceAssets/models/',
      );
    } else if (Platform.isAndroid) {
      // Android: Download GGUF format (already have, but check for updates)
      await downloadFromR2(
        remotePath: 'alice-assets/models/alice-phi3-q4.gguf',
        localPath: 'AliceAssets/models/',
      );
    }
  }

  Future<void> downloadLoRAAdapter(LoRAKind kind, {String? trainerId}) async {
    final String platformDir = Platform.isIOS ? 'ios' : 'android';
    final String adapterFormat = Platform.isIOS ? 'safetensors' : 'gguf';
    final String adapterName = _getAdapterName(kind, trainerId: trainerId);

    // Check for updates (compare checksums, not nightly)
    final String remotePath = 'alice-assets/adapters/$platformDir/$adapterName.$adapterFormat';

    // Only download if version/checksum changed
    if (await _needsUpdate(adapterName, adapterFormat)) {
      await downloadFromR2(
        remotePath: remotePath,
        localPath: 'AliceAssets/adapters/',
      );
    }
  }

  String _getAdapterName(LoRAKind kind, {String? trainerId}) {
    switch (kind) {
      case LoRAKind.U:
        return 'user_lora';
      case LoRAKind.T:
        return '${trainerId}_lora';
      case LoRAKind.GU:
        return 'global_user_lora';
      case LoRAKind.GT:
        return 'global_trainer_lora';
    }
  }

  // Check if adapter needs update (compare checksums, not nightly)
  Future<bool> _needsUpdate(String adapterName, String format) async {
    // Get local checksum
    final localChecksum = await _getLocalChecksum(adapterName, format);

    // Get server checksum (from manifest or metadata)
    final serverChecksum = await _getServerChecksum(adapterName, format);

    return localChecksum != serverChecksum;
  }
}
```

---

## Benefits

1. ✅ **No nightly server calls**: Device only checks for updates on app launch/background
2. ✅ **No on-device conversion**: Server converts once when adapter updates, device downloads pre-converted format
3. ✅ **Platform-optimized**: Each platform gets its native format
4. ✅ **Efficient**: No conversion overhead on device
5. ✅ **Scalable**: Works for any number of adapters
6. ✅ **Simple**: Keep existing llama.cpp for Android, add MLX for iOS

---

## Implementation Timeline

### Phase 1: Server-Side Conversion (Week 1)

- [x] Create `export_adapters_for_platforms.py` ✅
- [ ] Set up conversion pipeline (safetensors → GGUF for Android)
- [ ] Upload both formats to R2
- [ ] Test conversion pipeline

### Phase 2: Download Manager Update (Week 1-2)

- [ ] Add platform detection
- [ ] Update download logic to fetch platform-specific formats
- [ ] Add checksum/version checking (not nightly, just on check)
- [ ] Test iOS download (MLX + safetensors)
- [ ] Test Android download (GGUF)

### Phase 3: Integration (Week 2-3)

- [ ] Update iOS inference to use MLX
- [ ] Keep Android inference with llama.cpp
- [ ] Test end-to-end on both platforms

**Total**: ~3 weeks

---

## Summary

**iOS**: MLX base model + safetensors LoRA adapters (no conversion)
**Android**: GGUF base model + GGUF LoRA adapters (server-side conversion, one-time when adapter updates)

**Key Benefits**:

1. ✅ **No nightly server calls**: Device only checks for updates on app launch/background
2. ✅ **No on-device conversion**: Server converts once when adapter updates, device downloads pre-converted format
3. ✅ **Platform-optimized**: Each platform gets its native format

**Workflow**:

1. Training completes → Produces `adapters.safetensors`
2. Server converts once (when adapter updates):
   - iOS: Copy safetensors (no conversion)
   - Android: Convert safetensors → GGUF (one-time)
3. Upload both formats to R2
4. Device checks for updates (on app launch/background, not nightly)
5. Device downloads appropriate format for its platform
6. **No nightly server calls needed!**

## Related

^[source-materials/mirrors/doctrine/EVOLoRA_Mesh_Dual_Model_Format_Plan.md]
