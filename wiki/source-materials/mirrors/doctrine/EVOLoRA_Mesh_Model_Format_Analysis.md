---
title: EVOLoRA_Mesh_Model_Format_Analysis
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_Model_Format_Analysis.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh Model Format Analysis

**Problem**: Different runtimes require different model formats. We can't use the same model for llama.cpp, MLX, and CoreML.

---

## Current Model Formats

### Base Model Formats

| Runtime       | Format                  | Current Status                              | LoRA Support                       |
| ------------- | ----------------------- | ------------------------------------------- | ---------------------------------- |
| **llama.cpp** | GGUF                    | ✅ `alice-phi3-q4.gguf` (2.2GB)             | ✅ GGUF LoRA (requires conversion) |
| **MLX**       | MLX (safetensors-based) | ✅ Training exists, inference format needed | ✅ Native safetensors              |
| **CoreML**    | .mlpackage              | ⚠️ Export script exists, not deployed       | ❓ LoRA support unclear            |

### LoRA Adapter Formats

| Runtime       | Format      | Conversion Needed                 |
| ------------- | ----------- | --------------------------------- |
| **llama.cpp** | GGUF        | ❌ Yes (safetensors → GGUF)       |
| **MLX**       | Safetensors | ✅ Native (no conversion)         |
| **CoreML**    | ❓ Unknown  | ❓ May need conversion or merging |

---

## Model Conversion Requirements

### For MLX Inference

**Base Model**:

- Current: `alice-phi3-q4.gguf` (llama.cpp format)
- Needed: MLX format (safetensors-based)
- Conversion: GGUF → HuggingFace → MLX

**LoRA Adapters**:

- Current: `adapters.safetensors` (from MLX training)
- Needed: Same (safetensors) ✅
- Conversion: None needed!

**Process**:

```python
# 1. Convert base model GGUF → HuggingFace (if needed)
# OR use existing HuggingFace model

# 2. Convert HuggingFace → MLX format
from mlx_lm import convert
convert("microsoft/Phi-3-mini-4k-instruct", "models/alice-phi3-mlx")

# 3. LoRA adapters are already in safetensors (from MLX training)
# No conversion needed!
```

### For CoreML Inference

**Base Model**:

- Current: `alice-phi3-q4.gguf` (llama.cpp format)
- Needed: `.mlpackage` (CoreML format)
- Conversion: GGUF → HuggingFace → CoreML

**LoRA Adapters**:

- Current: `adapters.safetensors` (from MLX training)
- Needed: ❓ Unknown (CoreML LoRA support unclear)
- Conversion: May need to merge into base model

**Process**:

```python
# 1. Convert base model GGUF → HuggingFace
# OR use existing HuggingFace model

# 2. Merge LoRA adapters into base model
from peft import PeftModel
model = AutoModelForCausalLM.from_pretrained("base_model")
model = PeftModel.from_pretrained(model, "lora_adapter")
model = model.merge_and_unload()  # Merge LoRA into base

# 3. Convert merged model → CoreML
python training/export_coreml.py --model-path merged_model
```

**Problem**: CoreML doesn't support dynamic LoRA loading (as far as we know). Would need to merge adapters into base model, losing dynamic switching capability.

---

## Runtime Decision Matrix

### Option 1: MLX (Recommended)

**Base Model Conversion**:

- ✅ One-time conversion: HuggingFace → MLX format
- ✅ LoRA adapters: No conversion (already safetensors)
- ✅ Dynamic LoRA: Supported natively
- ✅ Incremental training: Supported natively

**Conversion Script Needed**:

```python
# training/export_mlx_base.py
from mlx_lm import convert

# Convert base model to MLX format
convert(
    "microsoft/Phi-3-mini-4k-instruct",
    "models/alice-phi3-mlx-base"
)
```

**Deployment**:

- Base model: MLX format (~2-3GB, can quantize)
- LoRA adapters: Safetensors (no conversion)
- Total conversion: One-time base model conversion

### Option 2: CoreML

**Base Model Conversion**:

- ✅ One-time conversion: HuggingFace → CoreML (.mlpackage)
- ❌ LoRA adapters: Must merge into base (loses dynamic switching)
- ❌ Dynamic LoRA: Not supported (need separate merged models)
- ❌ Incremental training: Not supported (CoreML is inference-only)

**Problem**: CoreML doesn't support dynamic LoRA adapters. You'd need:

- Separate merged models for each adapter combination
- Or merge adapters server-side, download pre-merged models
- **Defeats EVOLoRA Mesh architecture** (can't switch adapters dynamically)

**Verdict**: ❌ **Not suitable** - Loses dynamic adapter switching

### Option 3: Keep llama.cpp + Accept Conversion

**Base Model**:

- ✅ Already have: `alice-phi3-q4.gguf`
- ✅ No conversion needed

**LoRA Adapters**:

- ❌ Must convert: Safetensors → GGUF (nightly)
- ❌ Conversion overhead: 1-2 minutes
- ❌ Can't continue training on GGUF

**Verdict**: ⚠️ **Works but doesn't scale** - Conversion overhead is the problem

---

## Recommended: MLX Migration

### Model Format Requirements

**Base Model**:

- Convert once: HuggingFace → MLX format
- Size: ~2-3GB (can quantize to reduce)
- Format: MLX safetensors (different from HuggingFace safetensors)

**LoRA Adapters**:

- Format: Safetensors (from MLX training) ✅
- No conversion needed ✅
- Can continue training incrementally ✅

### Conversion Pipeline

**Step 1: Convert Base Model (One-Time)**

```python
# training/export_mlx_base.py
from mlx_lm import convert, quantize

# Convert HuggingFace → MLX
convert(
    "microsoft/Phi-3-mini-4k-instruct",
    "models/alice-phi3-mlx-base"
)

# Optional: Quantize for smaller size
quantize(
    "models/alice-phi3-mlx-base",
    "models/alice-phi3-mlx-base-q4",
    q_bits=4
)
```

**Step 2: LoRA Adapters (No Conversion)**

- MLX training already produces safetensors
- Use directly for inference
- Continue training incrementally

### Model Storage

**R2 Storage**:

```
alice-assets/
├── models/
│   ├── alice-phi3-mlx-base/          ← MLX base model (one-time conversion)
│   │   ├── model.safetensors
│   │   ├── config.json
│   │   └── tokenizer.json
│   └── alice-phi3-q4.gguf            ← Keep for llama.cpp fallback
├── adapters/
│   ├── user/user_lora.safetensors    ← MLX format (no conversion)
│   ├── trainer/{trainerId}_lora.safetensors
│   └── global/
│       ├── global_user_lora.safetensors
│       └── global_trainer_lora.safetensors
```

---

## Migration Impact

### What Changes

1. **Base Model**: Need MLX format (one-time conversion)
2. **Inference Engine**: Switch from llama.cpp to MLX
3. **LoRA Adapters**: No change (already safetensors from MLX training)

### What Stays the Same

1. **Training Pipeline**: Already using MLX ✅
2. **LoRA Adapter Format**: Already safetensors ✅
3. **Incremental Training**: Works natively with MLX ✅

---

## Implementation Steps

### Phase 1: Convert Base Model (1 day)

- [ ] Create `export_mlx_base.py` script
- [ ] Convert HuggingFace model → MLX format
- [ ] Quantize if needed (reduce size)
- [ ] Upload to R2

### Phase 2: Update Download Manager (1 day)

- [ ] Add MLX base model to `AliceAssetDownloadManager`
- [ ] Download MLX format instead of GGUF
- [ ] Update model path references

### Phase 3: Build MLX Inference Engine (2-3 weeks)

- [ ] Add Python runtime to iOS
- [ ] Create MLX inference script
- [ ] Create Swift wrapper
- [ ] Test inference performance

### Phase 4: Migrate Inference (1 week)

- [ ] Replace llama.cpp calls with MLX
- [ ] Update Flutter integration
- [ ] Test end-to-end

**Total**: ~4-5 weeks

---

## Model Size Comparison

| Format            | Size   | Quantization    |
| ----------------- | ------ | --------------- |
| **GGUF (Q4_K_M)** | 2.2GB  | 4-bit quantized |
| **MLX (FP16)**    | ~7GB   | 16-bit float    |
| **MLX (Q4)**      | ~2GB   | 4-bit quantized |
| **CoreML (int8)** | ~3-4GB | 8-bit quantized |

**Recommendation**: Use MLX with Q4 quantization to match GGUF size (~2GB).

---

## Summary

**For MLX**:

- ✅ Base model: One-time conversion (HuggingFace → MLX) - **Script created**: `export_mlx_base.py`
- ✅ LoRA adapters: No conversion (already safetensors from MLX training)
- ✅ Dynamic switching: Supported
- ✅ Incremental training: Supported

**For CoreML**:

- ✅ Base model: One-time conversion (HuggingFace → CoreML) - **Script exists**: `export_coreml.py`
- ❌ LoRA adapters: Must merge (loses dynamic switching)
- ❌ Not suitable for EVOLoRA Mesh

**Recommendation**: **MLX** - Only requires base model conversion (one-time), adapters work natively.

**Action Items**:

1. Run `python training/export_mlx_base.py` to convert base model
2. Upload MLX base model to R2
3. Build MLX inference engine for iOS
4. Update download manager to fetch MLX format

## Related
