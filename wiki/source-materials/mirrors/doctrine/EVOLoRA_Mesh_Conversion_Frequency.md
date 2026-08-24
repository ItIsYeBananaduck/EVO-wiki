---
title: EVOLoRA_Mesh_Conversion_Frequency
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_Conversion_Frequency.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh Conversion Frequency & Update Strategy

**Problem**: Android needs GGUF conversion, which could delay updates compared to iOS (which uses safetensors directly).

**Solution**: Server-side conversion happens **immediately** when adapters are updated, ensuring both platforms get updates at the same frequency.

---

## Update Frequency by Adapter Type

### User (U) & Trainer (T) Adapters

**Training Frequency**: Daily (nightly on-device training)

**Update Flow**:

1. **Nightly (3 AM)**: On-device training completes → Produces `user_lora.safetensors`
2. **Immediately**: Upload safetensors to server
3. **Server converts** (parallel, <1 min):
   - iOS: Copy safetensors (no conversion)
   - Android: Convert safetensors → GGUF
4. **Upload to R2** (immediately after conversion)
5. **Device checks** (on app launch/background):
   - Both platforms see update available
   - Download appropriate format
   - **Both get daily updates** ✅

**Timeline**:

```
3:00 AM - Training completes (safetensors)
3:00 AM - Upload to server
3:01 AM - Server converts (iOS: copy, Android: GGUF conversion)
3:02 AM - Upload to R2
3:02 AM - Both formats available
Next app launch - Device downloads update
```

### Global User (GU) & Global Trainer (GT) Adapters

**Training Frequency**: Weekly (federated aggregation)

**Update Flow**:

1. **Weekly (Sunday night)**: Federated aggregation completes → Produces `global_user_lora.safetensors`
2. **Immediately**: Server converts (parallel):
   - iOS: Copy safetensors (no conversion)
   - Android: Convert safetensors → GGUF
3. **Upload to R2** (immediately after conversion)
4. **Device checks** (on app launch/background):
   - Both platforms see update available
   - Download appropriate format
   - **Both get weekly updates** ✅

---

## Server-Side Conversion Pipeline

### Automated Conversion Service

**File**: `federated-server/src/services/adapter_converter.py` (NEW)

```python
"""
Automated adapter conversion service.
Runs immediately when adapters are updated.
"""

import asyncio
from pathlib import Path
import subprocess

async def convert_adapter_for_platforms(
    safetensors_path: Path,
    base_model_path: Path,
    adapter_name: str,
    adapter_kind: str  # 'user', 'trainer', 'global_user', 'global_trainer'
):
    """
    Convert adapter immediately when updated.
    Runs in parallel for both platforms.
    """
    output_dir = Path("adapters") / adapter_kind

    # Parallel conversion for both platforms
    tasks = [
        _convert_for_ios(safetensors_path, output_dir, adapter_name),
        _convert_for_android(safetensors_path, base_model_path, output_dir, adapter_name),
    ]

    ios_path, android_path = await asyncio.gather(*tasks)

    # Upload both to R2 immediately
    await asyncio.gather(
        upload_to_r2(ios_path, f"alice-assets/adapters/ios/{adapter_name}.safetensors"),
        upload_to_r2(android_path, f"alice-assets/adapters/android/{adapter_name}.gguf"),
    )

    # Update manifest with checksums and timestamp
    await update_manifest(adapter_name, ios_path, android_path)

    print(f"✅ Converted and uploaded {adapter_name} for both platforms")

async def _convert_for_ios(safetensors_path: Path, output_dir: Path, adapter_name: str):
    """Copy safetensors (no conversion needed for iOS)."""
    ios_path = output_dir / "ios" / f"{adapter_name}.safetensors"
    ios_path.parent.mkdir(parents=True, exist_ok=True)
    shutil.copy(safetensors_path, ios_path)
    return ios_path

async def _convert_for_android(
    safetensors_path: Path,
    base_model_path: Path,
    output_dir: Path,
    adapter_name: str
):
    """Convert safetensors → GGUF for Android."""
    android_path = output_dir / "android" / f"{adapter_name}.gguf"
    android_path.parent.mkdir(parents=True, exist_ok=True)

    # Use llama.cpp's convert-lora-to-gguf (fast, <1 min)
    subprocess.run([
        "python", "alicecore_build/llama.cpp/convert_lora_to_gguf.py",
        "--outfile", str(android_path),
        "--base", str(base_model_path),
        str(safetensors_path)
    ], check=True)

    return android_path
```

### Integration Points

#### 1. On-Device Training Completion (U/T Adapters)

**File**: `app/src/lib/services/ml/qloraTrainer.ts` (MODIFY)

```typescript
async function runDailyTraining() {
  // ... training logic ...

  // After training completes
  const adapterPath = await saveAdapter("user_lora.safetensors");

  // Upload to server immediately
  await uploadAdapterToServer(adapterPath, "user_lora");

  // Server will convert and upload to R2 automatically
  // Device will see update on next check
}
```

#### 2. Federated Aggregation Completion (GU/GT Adapters)

**File**: `federated-server/src/ml/gguf_merger.py` (MODIFY)

```python
async def aggregate_and_convert():
    # ... aggregation logic ...

    # After aggregation completes
    safetensors_path = await train_global_adapter()  # Produces safetensors

    # Convert immediately for both platforms
    await convert_adapter_for_platforms(
        safetensors_path=safetensors_path,
        base_model_path=Path("models/alice-phi3-q4.gguf"),
        adapter_name="global_user_lora",
        adapter_kind="global"
    )

    # Both formats now available on R2
```

---

## Update Frequency Comparison

| Adapter Type            | Training Frequency | iOS Update | Android Update | Sync?  |
| ----------------------- | ------------------ | ---------- | -------------- | ------ |
| **U (User)**            | Daily (nightly)    | Daily ✅   | Daily ✅       | ✅ Yes |
| **T (Trainer)**         | Daily (nightly)    | Daily ✅   | Daily ✅       | ✅ Yes |
| **GU (Global User)**    | Weekly             | Weekly ✅  | Weekly ✅      | ✅ Yes |
| **GT (Global Trainer)** | Weekly             | Weekly ✅  | Weekly ✅      | ✅ Yes |

**Result**: Both platforms get updates at the **same frequency** because:

1. Server converts immediately when adapter is updated
2. Conversion is fast (<1 min for GGUF)
3. Both formats uploaded to R2 immediately
4. Devices check on app launch/background (not scheduled)

---

## Conversion Performance

### Conversion Time

| Platform    | Format      | Conversion Time | Notes                |
| ----------- | ----------- | --------------- | -------------------- |
| **iOS**     | Safetensors | <1 second       | Just copy file       |
| **Android** | GGUF        | ~30-60 seconds  | llama.cpp conversion |

**Total**: Both formats available within ~1 minute of training completion.

### Optimization

**Parallel Conversion**: Convert for both platforms simultaneously (not sequential)

**Caching**: Skip conversion if adapter unchanged (checksum comparison)

**Background Processing**: Conversion happens in background, doesn't block training completion

---

## Device Update Check Strategy

### Not Scheduled (No Nightly Calls)

**Check Triggers**:

1. **App launch**: Check for adapter updates
2. **Background refresh**: When app enters foreground
3. **Manual refresh**: User-initiated check

**Check Process**:

```dart
Future<void> checkForAdapterUpdates() async {
  // Get local checksums
  final localChecksums = await getLocalAdapterChecksums();

  // Get server manifest (lightweight, just checksums + timestamps)
  final serverManifest = await fetchAdapterManifest();

  // Compare and download if needed
  for (final adapter in adapters) {
    if (localChecksums[adapter] != serverManifest[adapter].checksum) {
      await downloadAdapter(adapter);  // Download platform-specific format
    }
  }
}
```

**Benefits**:

- ✅ No scheduled nightly calls
- ✅ Lightweight manifest check (just checksums)
- ✅ Only downloads when update available
- ✅ Works offline (checks when online)

---

## Summary

**Key Points**:

1. ✅ **Both platforms get updates at same frequency**: Daily for U/T, weekly for GU/GT
2. ✅ **Server converts immediately**: No delay between training and availability
3. ✅ **Fast conversion**: GGUF conversion takes <1 min
4. ✅ **No nightly calls**: Device checks on app launch/background
5. ✅ **Parallel processing**: Both formats converted simultaneously

**Result**: Android users get **daily updates** just like iOS users, not weekly. The conversion happens server-side immediately when adapters are updated, ensuring both platforms stay in sync.

## Related
