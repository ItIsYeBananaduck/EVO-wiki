---
title: EVOLoRA_Mesh_OnDevice_Conversion_Plan
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVOLoRA_Mesh_OnDevice_Conversion_Plan.md
updated: 2026-07-24
---

# EVOLoRA Mesh On-Device Conversion Plan

**Principle**: Keep everything on-device. Only weekly aggregation moves to server.

**Strategy**:

- **U/T adapters**: Convert on-device immediately after training (no server calls)
- **GU/GT adapters**: Convert server-side during weekly aggregation (only server-side operation)

---

## On-Device Conversion for U/T Adapters

### User (U) & Trainer (T) Adapters

**Training**: Daily (nightly on-device training)

**Process** (All On-Device):

1. **Nightly (3 AM)**: On-device training completes
   - Produces: `user_lora.safetensors` (MLX format)
   - Location: `AliceAssets/adapters/user/user_lora.safetensors`

2. **On-device conversion** (immediately after training):
   - **iOS**: Use safetensors directly (no conversion needed) ✅
   - **Android**: Convert safetensors → GGUF on-device
     - Use llama.cpp's `convert-lora-to-gguf` (native binding)
     - Output: `AliceAssets/adapters/user/user_lora.gguf`
     - Time: ~30-60 seconds (acceptable for nightly background task)

3. **Store locally**:
   - iOS: `user_lora.safetensors` (ready to use)
   - Android: `user_lora.gguf` (ready to use)

4. **Upload delta to server** (for aggregation only):
   - Upload safetensors delta to server (for weekly GU/GT aggregation)
   - **Not for conversion** - conversion already done on-device

**Key**: No server calls for conversion. Everything happens on-device.

---

## Android On-Device Conversion Implementation

### Native Binding for llama.cpp Conversion

**File**: `flutter_app/android/app/src/main/cpp/convert_lora_adapter.cpp` (NEW)

```cpp
#include <jni.h>
#include <string>

// llama.cpp headers
extern "C" {
    // llama.cpp's convert-lora-to-gguf functionality
    int convert_lora_to_gguf(
        const char* safetensors_path,
        const char* base_model_path,
        const char* output_gguf_path
    );
}

extern "C" JNIEXPORT jint JNICALL
Java_com_example_flutter_app_LlamaPlugin_convertLoRAToGGUF(
    JNIEnv *env,
    jobject thiz,
    jstring safetensors_path,
    jstring base_model_path,
    jstring output_gguf_path
) {
    const char* safetensors = env->GetStringUTFChars(safetensors_path, nullptr);
    const char* base_model = env->GetStringUTFChars(base_model_path, nullptr);
    const char* output = env->GetStringUTFChars(output_gguf_path, nullptr);

    int result = convert_lora_to_gguf(safetensors, base_model, output);

    env->ReleaseStringUTFChars(safetensors_path, safetensors);
    env->ReleaseStringUTFChars(base_model_path, base_model);
    env->ReleaseStringUTFChars(output_gguf_path, output);

    return result; // 0 on success
}
```

### Kotlin Wrapper

**File**: `flutter_app/android/app/src/main/kotlin/com/example/flutter_app/LlamaPlugin.kt` (MODIFY)

```kotlin
class LlamaPlugin {
    // Native method for converting safetensors → GGUF
    private external fun convertLoRAToGGUF(
        safetensorsPath: String,
        baseModelPath: String,
        outputGGUFPath: String
    ): Int

    /**
     * Convert LoRA adapter from safetensors to GGUF format.
     * Called immediately after on-device training completes.
     */
    fun convertAdapterToGGUF(
        safetensorsPath: String,
        baseModelPath: String,
        outputGGUFPath: String
    ): Boolean {
        val result = convertLoRAToGGUF(safetensorsPath, baseModelPath, outputGGUFPath)
        return result == 0
    }
}
```

### Integration with Training

**File**: `app/src/lib/services/ml/qloraTrainer.ts` (MODIFY)

```typescript
async runDailyTraining(): Promise<TrainingSession> {
    // ... existing training logic ...

    // After training completes
    const safetensorsPath = await saveAdapter('user_lora.safetensors');

    // On-device conversion for Android
    if (Platform.isAndroid) {
        const baseModelPath = await getBaseModelPath(); // alice-phi3-q4.gguf
        const ggufPath = 'AliceAssets/adapters/user/user_lora.gguf';

        // Convert on-device (no server call)
        await LlamaPlugin.convertLoRAToGGUF(
            safetensorsPath,
            baseModelPath,
            ggufPath
        );

        console.log('✅ Converted adapter to GGUF on-device');
    }

    // iOS: Use safetensors directly (no conversion)
    // Android: Now has GGUF ready to use

    // Upload delta to server (for aggregation only, not for conversion)
    await federatedUploader.enqueueDelta(safetensorsPath);

    return result;
}
```

---

## Server-Side Conversion (GU/GT Adapters Only)

### Global User (GU) & Global Trainer (GT) Adapters

**Training**: Weekly (federated aggregation on server)

**Process** (Server-Side, Only Weekly Operation):

1. **Weekly (Sunday night)**: Federated aggregation completes
   - Server produces: `global_user_lora.safetensors`

2. **Server-side conversion** (immediately after aggregation):
   - **iOS**: Copy safetensors (no conversion) → Upload to R2
   - **Android**: Convert safetensors → GGUF → Upload to R2
   - Use: `training/export_adapters_for_platforms.py`

3. **Upload to R2**:
   - Upload both formats to R2
   - Update manifest with checksums/versions

4. **Device downloads** (on app launch/background):
   - Compare local checksum with server checksum
   - Download if new version available
   - **Both platforms get weekly updates** ✅

**Key**: Only GU/GT adapters use server-side conversion (weekly aggregation). This is the only server-side operation.

---

## Update Frequency

| Adapter Type            | Training          | Conversion                  | Server Calls                             |
| ----------------------- | ----------------- | --------------------------- | ---------------------------------------- |
| **U (User)**            | On-device (daily) | On-device (immediate)       | None (only delta upload for aggregation) |
| **T (Trainer)**         | On-device (daily) | On-device (immediate)       | None (only delta upload for aggregation) |
| **GU (Global User)**    | Server (weekly)   | Server (during aggregation) | Weekly (aggregation only)                |
| **GT (Global Trainer)** | Server (weekly)   | Server (during aggregation) | Weekly (aggregation only)                |

**Result**:

- ✅ No unnecessary server calls for U/T adapters
- ✅ Only weekly server operation (GU/GT aggregation)
- ✅ Both platforms get updates at same frequency

---

## Implementation Timeline

### Phase 1: On-Device Conversion (Week 1-2)

- [ ] Build llama.cpp convert-lora-to-gguf into Android app
- [ ] Create JNI wrapper for conversion
- [ ] Integrate with training completion
- [ ] Test on-device conversion performance

### Phase 2: Server-Side Conversion (Week 2)

- [ ] Set up conversion pipeline for GU/GT adapters
- [ ] Integrate with federated aggregation
- [ ] Test server-side conversion

### Phase 3: Integration (Week 3)

- [ ] Test end-to-end on both platforms
- [ ] Verify no unnecessary server calls
- [ ] Performance testing

**Total**: ~3 weeks

---

## Summary

**U/T Adapters**:

- ✅ Train on-device (daily)
- ✅ Convert on-device (immediately after training)
- ✅ No server calls for conversion
- ✅ Upload delta to server (for aggregation only)

**GU/GT Adapters**:

- ✅ Aggregate on server (weekly)
- ✅ Convert on server (during aggregation)
- ✅ Upload to R2
- ✅ Device downloads on app launch/background

**Key Principle**: Keep everything on-device. Only weekly aggregation moves to server.

## Related

^[source-materials/mirrors/doctrine/EVOLoRA_Mesh_OnDevice_Conversion_Plan.md]
