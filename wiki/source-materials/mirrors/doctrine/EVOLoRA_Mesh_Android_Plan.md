---
title: EVOLoRA_Mesh_Android_Plan
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_Android_Plan.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh Android Implementation Plan

**Problem**: MLX is Apple-only. Android needs a different runtime for EVOLoRA Mesh.

**Current State**: Android uses llama.cpp (same as iOS), but with the same conversion problem.

---

## Android Runtime Options

### Option 1: llama.cpp (Current - Keep)

**Pros**:

- ✅ Already integrated (`LlamaPlugin.kt` exists)
- ✅ Same codebase as iOS
- ✅ Proven performance
- ✅ GPU acceleration (Vulkan/OpenCL)

**Cons**:

- ❌ Same conversion problem (safetensors → GGUF)
- ❌ No incremental training support
- ❌ Conversion overhead

**LoRA Support**:

- ✅ llama.cpp supports GGUF LoRAs
- ❌ Requires safetensors → GGUF conversion

**Verdict**: ⚠️ **Keep but optimize conversion** - Same issues as iOS

---

### Option 2: ONNX Runtime

**Pros**:

- ✅ Cross-platform (iOS + Android)
- ✅ Good performance
- ✅ LoRA support (via extensions)
- ✅ No conversion needed (can use ONNX format)

**Cons**:

- ⚠️ LoRA support is limited (may need merging)
- ⚠️ Model conversion required (HuggingFace → ONNX)
- ⚠️ Incremental training unclear

**LoRA Support**:

- ⚠️ Limited - may need to merge adapters into base model
- ⚠️ Dynamic switching may not be fully supported

**Verdict**: ⚠️ **Possible but complex** - LoRA support unclear

---

### Option 3: Transformers.js (Web-based)

**Pros**:

- ✅ Native safetensors support
- ✅ LoRA support
- ✅ No conversion needed

**Cons**:

- ❌ Web-only (not native Android)
- ❌ Performance overhead
- ❌ Not suitable for mobile

**Verdict**: ❌ **Not applicable** - Web-only

---

### Option 4: TensorFlow Lite

**Pros**:

- ✅ Native Android support
- ✅ Good performance
- ✅ Quantization support

**Cons**:

- ❌ LoRA support unclear
- ❌ Model conversion required
- ❌ May need to merge adapters

**Verdict**: ⚠️ **Unclear** - Need to verify LoRA support

---

### Option 5: Hybrid - llama.cpp with Optimized Conversion

**Approach**: Keep llama.cpp, but optimize the conversion process

**Optimizations**:

1. **Lazy conversion**: Only convert when adapter changes
2. **Incremental conversion**: Only convert changed weights
3. **Background conversion**: Lower priority, non-blocking
4. **Cache converted adapters**: Don't reconvert if unchanged

**Pros**:

- ✅ Keep existing llama.cpp integration
- ✅ Proven performance
- ✅ Minimal migration risk

**Cons**:

- ⚠️ Still requires conversion (but optimized)
- ⚠️ Doesn't solve incremental training

**Verdict**: ✅ **Recommended** - Best balance of risk/benefit

---

## Recommended: Keep llama.cpp + Optimize Conversion

### Why Keep llama.cpp?

1. **Already integrated**: `LlamaPlugin.kt` exists
2. **Proven performance**: Works well on Android
3. **Same codebase**: Can share logic with iOS
4. **GPU acceleration**: Vulkan/OpenCL support

### Conversion Optimization Strategy

#### 1. Server-Side Conversion (for GU/GT Adapters)

**Process**:

- Train GU/GT adapters on server (safetensors)
- Convert to GGUF server-side (one-time)
- Upload GGUF to R2
- Download GGUF directly on device

**Benefit**: No on-device conversion for global adapters

#### 2. On-Device Conversion (for U/T Adapters)

**Process**:

- Train U/T adapters on-device (safetensors)
- Convert to GGUF on-device (optimized)
- Cache converted adapter
- Only reconvert if adapter changes

**Optimizations**:

- **Lazy conversion**: Only convert when needed
- **Incremental conversion**: Only convert changed weights (if possible)
- **Background conversion**: Run in background task
- **Cache checksum**: Skip conversion if adapter unchanged

#### 3. Conversion Script (Android)

**File**: `flutter_app/android/app/src/main/cpp/convert_lora_to_gguf.cpp` (NEW)

```cpp
// Native C++ conversion using llama.cpp's convert-lora-to-gguf
// Called from Kotlin via JNI

extern "C" JNIEXPORT jint JNICALL
Java_com_example_flutter_app_LlamaPlugin_convertLoRAToGGUF(
    JNIEnv *env,
    jobject thiz,
    jstring safetensors_path,
    jstring gguf_output_path,
    jstring base_model_path
) {
    // Call llama.cpp's convert-lora-to-gguf utility
    // This requires linking llama.cpp library
    // Return 0 on success, non-zero on error
}
```

**Kotlin Wrapper**:

```kotlin
// In LlamaPlugin.kt
private external fun convertLoRAToGGUF(
    safetensorsPath: String,
    ggufOutputPath: String,
    baseModelPath: String
): Int

fun convertAdapterToGGUF(
    safetensorsPath: String,
    ggufOutputPath: String
): Boolean {
    val baseModelPath = getModelPath() // Get base GGUF model path
    val result = convertLoRAToGGUF(safetensorsPath, ggufOutputPath, baseModelPath)
    return result == 0
}
```

---

## Android Implementation Plan

### Phase 1: Optimize Conversion (Week 1-2)

#### 1.1 Server-Side Conversion for GU/GT

- [ ] Add conversion step to federated server pipeline
- [ ] Convert GU/GT safetensors → GGUF after training
- [ ] Upload GGUF to R2
- [ ] Update download manager to fetch GGUF directly

#### 1.2 On-Device Conversion for U/T

- [ ] Build llama.cpp's `convert-lora-to-gguf` into Android app
- [ ] Create JNI wrapper for conversion
- [ ] Add lazy conversion logic (only when adapter changes)
- [ ] Add checksum caching (skip if unchanged)
- [ ] Run conversion in background task

#### 1.3 Conversion Optimization

- [ ] Implement incremental conversion (if possible)
- [ ] Add progress callbacks
- [ ] Handle conversion errors gracefully
- [ ] Cache converted adapters

### Phase 2: LoRA Loading (Week 2-3)

#### 2.1 llama.cpp LoRA API

- [ ] Verify llama.cpp LoRA API works on Android
- [ ] Test loading multiple adapters
- [ ] Test adapter switching
- [ ] Measure performance impact

#### 2.2 Integration with MeshRouter

- [ ] Update `LlamaPlugin.kt` to accept adapter stack
- [ ] Load adapters from `MeshRouter.buildAdapterStack()`
- [ ] Apply adapters with scales
- [ ] Test end-to-end

### Phase 3: Testing (Week 3-4)

#### 3.1 Performance Testing

- [ ] Measure conversion time (should be <2 min)
- [ ] Measure inference latency with adapters
- [ ] Test memory usage
- [ ] Test battery impact

#### 3.2 Integration Testing

- [ ] Test U adapter (user-specific)
- [ ] Test T adapter (trainer-specific)
- [ ] Test GU/GT adapters (global)
- [ ] Test adapter switching
- [ ] Test error handling

---

## Alternative: ONNX Runtime (If llama.cpp Doesn't Work)

### ONNX Conversion Pipeline

**Base Model**:

```python
# training/export_onnx.py
from transformers import AutoModelForCausalLM
import onnxruntime as ort

# Convert HuggingFace → ONNX
model = AutoModelForCausalLM.from_pretrained("microsoft/Phi-3-mini-4k-instruct")
# Export to ONNX format
# ...
```

**LoRA Adapters**:

- Option A: Merge into base model (loses dynamic switching)
- Option B: Use ONNX LoRA extension (if available)
- Option C: Multiple ONNX models (one per adapter combination)

**Verdict**: ⚠️ **Complex** - Only if llama.cpp fails

---

## Comparison: iOS vs Android

| Feature                  | iOS (MLX)                           | Android (llama.cpp)        |
| ------------------------ | ----------------------------------- | -------------------------- |
| **Base Model**           | MLX format (one-time conversion)    | GGUF (already have)        |
| **LoRA Adapters**        | Safetensors (native, no conversion) | GGUF (requires conversion) |
| **Conversion**           | None needed                         | On-device (optimized)      |
| **Incremental Training** | ✅ Supported                        | ❌ Not supported           |
| **Dynamic Switching**    | ✅ Supported                        | ✅ Supported               |
| **Performance**          | Fast (Metal)                        | Fast (Vulkan/OpenCL)       |

**Key Difference**: iOS can eliminate conversion entirely with MLX. Android must optimize conversion but can't eliminate it.

---

## Recommended Approach

### For Android: Keep llama.cpp + Optimize Conversion

**Why**:

1. Already integrated
2. Proven performance
3. Minimal migration risk
4. Can share code with iOS (where applicable)

**Optimizations**:

1. Server-side conversion for GU/GT (no on-device conversion)
2. Lazy on-device conversion for U/T (only when needed)
3. Checksum caching (skip if unchanged)
4. Background conversion (non-blocking)

**Trade-off**: Accept optimized conversion overhead (~1-2 min nightly) for U/T adapters, but eliminate it for GU/GT.

---

## Implementation Timeline

### Week 1-2: Conversion Optimization

- Server-side conversion for GU/GT
- On-device conversion for U/T
- Lazy conversion logic
- Checksum caching

### Week 3: LoRA Integration

- llama.cpp LoRA API on Android
- MeshRouter integration
- Adapter loading/application

### Week 4: Testing

- Performance testing
- Integration testing
- Error handling

**Total**: ~4 weeks

---

## Next Steps

1. **Verify llama.cpp LoRA API on Android** - Test if it works
2. **Build conversion utility** - Integrate `convert-lora-to-gguf` into Android
3. **Implement lazy conversion** - Only convert when needed
4. **Test performance** - Ensure conversion is fast enough

**If llama.cpp LoRA doesn't work on Android**:

- Fallback to ONNX Runtime
- Or merge adapters server-side (loses dynamic switching)

## Related
