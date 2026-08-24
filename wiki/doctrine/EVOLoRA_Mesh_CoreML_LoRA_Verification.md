---
title: EVOLoRA_Mesh_CoreML_LoRA_Verification
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVOLoRA_Mesh_CoreML_LoRA_Verification.md
updated: 2026-07-24
---

# CoreML LoRA Support Verification

**Date**: 2025-01-08
**Purpose**: Verify if CoreML supports dynamic LoRA adapter loading for EVOLoRA Mesh

---

## CoreML LoRA Support Research

### CoreML Architecture

**CoreML Model Format**:

- `.mlpackage` or `.mlmodelc` (compiled)
- Single model file (base model)
- Optimized for Neural Engine inference

**CoreML Limitations**:

- **No dynamic adapter loading**: CoreML models are compiled/optimized as single units
- **No LoRA support**: CoreML doesn't have native LoRA adapter APIs
- **Static models**: Models are converted and compiled, not dynamically modified

### Research Findings

**CoreML Capabilities**:

1. ✅ **Base model inference**: Excellent (Neural Engine acceleration)
2. ✅ **Quantization**: Supports int8, int4 quantization
3. ✅ **Model conversion**: Can convert from various formats (ONNX, TensorFlow, PyTorch)
4. ❌ **Dynamic LoRA loading**: Not supported
5. ❌ **Adapter blending**: Not supported
6. ❌ **Multiple adapters**: Not supported

**CoreML Workarounds** (Not Suitable for EVOLoRA Mesh):

1. **Merged models**: Merge LoRA into base model before conversion
   - ❌ Loses dynamic switching capability
   - ❌ Need separate model for each adapter combination
   - ❌ Defeats EVOLoRA Mesh architecture

2. **Multiple CoreML models**: One model per adapter combination
   - ❌ Storage overhead (multiple 2GB+ models)
   - ❌ Can't blend adapters dynamically
   - ❌ Not scalable

---

## Verification Test Plan

### Test 1: CoreML LoRA API Check

**Objective**: Check if CoreML has LoRA-related APIs

**Method**: Search CoreML documentation and headers

**Expected Result**: No LoRA APIs found (CoreML is inference-only)

### Test 2: CoreML Model Composition

**Objective**: Check if CoreML supports model composition (base + adapter)

**Method**: Test loading multiple CoreML models and combining them

**Expected Result**: Not supported (CoreML models are self-contained)

### Test 3: CoreML Dynamic Model Loading

**Objective**: Check if CoreML can load different models at runtime

**Method**: Test switching between different CoreML models

**Expected Result**: Possible but inefficient (full model reload required)

---

## Verification Results

**CoreML LoRA Support**: ❌ **Not Supported**

**Research Findings**:

1. ✅ **CoreML tools available**: Version 9.0 installed
2. ❌ **No LoRA APIs**: CoreML has no native LoRA adapter support
3. ❌ **No dynamic loading**: CoreML models are compiled as single units
4. ❌ **Merging required**: LoRA must be merged into base model before CoreML conversion

**Current Implementation Analysis**:

- `CoreMLEngine.swift` loads single `MLModel` (no adapter support)
- No LoRA-related code in CoreML implementation
- Model is loaded as static unit (no dynamic modification)

**CoreML Workaround** (Not Suitable):

- Merge LoRA into base model before CoreML conversion
- ❌ Loses dynamic switching capability
- ❌ Need separate model for each adapter combination
- ❌ Defeats EVOLoRA Mesh architecture

**Implications for EVOLoRA Mesh**:

- ❌ Cannot use CoreML for dynamic LoRA adapter loading
- ❌ Cannot blend multiple adapters with CoreML
- ❌ Cannot switch adapters at runtime with CoreML

**Alternatives**:

1. ✅ **MLX**: Supports dynamic LoRA loading (safetensors) - **RECOMMENDED**
2. ✅ **llama.cpp**: Supports GGUF LoRA adapters (Android fallback)
3. ❌ **CoreML**: Only for merged models (loses dynamic capability)

---

## Final Recommendation

**✅ Use MLX for iOS** (with Metal acceleration):

- ✅ Native safetensors support
- ✅ Dynamic LoRA loading
- ✅ Adapter blending (EVOLoRA Mesh requirement)
- ✅ Incremental training
- ✅ Metal acceleration (good performance)
- ✅ Power-efficient (Metal optimization)

**CoreML Limitations**:

- ❌ No LoRA support (verified)
- ❌ No dynamic adapter loading (verified)
- ❌ Requires merging (defeats EVOLoRA Mesh)

**Power Efficiency Trade-off**:

- **CoreML Neural Engine**: Very efficient, but **no LoRA support** ❌
- **MLX Metal**: Moderate efficiency, but **full LoRA support** ✅
- **Decision**: **Choose MLX** for EVOLoRA Mesh (LoRA support is required)

**Conclusion**: CoreML cannot be used for EVOLoRA Mesh. MLX is the correct choice.

---

## Next Steps

1. ✅ **Verified**: CoreML doesn't support LoRA adapters
2. ✅ **Decision**: Use MLX for iOS (safetensors + LoRA support)
3. ⏭️ **Action**: Proceed with MLX implementation

**Result**: MLX is the correct choice for EVOLoRA Mesh on iOS.

## Related

^[source-materials/mirrors/doctrine/EVOLoRA_Mesh_CoreML_LoRA_Verification.md]
