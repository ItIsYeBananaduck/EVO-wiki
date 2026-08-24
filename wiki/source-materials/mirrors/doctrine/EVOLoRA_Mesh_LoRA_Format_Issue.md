---
title: EVOLoRA_Mesh_LoRA_Format_Issue
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_LoRA_Format_Issue.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh LoRA Format Compatibility Issue

**Problem**: llama.cpp only supports **GGUF format** for LoRA adapters, but our training pipeline produces **safetensors** format (standard HuggingFace/PEFT).

---

## The Problem

### Current Training Pipeline Output

- **Format**: `adapter_model.safetensors` (HuggingFace/PEFT standard)
- **Location**: `training/alice-3b-lora/adapter_model.safetensors`
- **Config**: `adapter_config.json` (LoRA rank, alpha, target modules)

### llama.cpp Requirements

- **Format**: GGUF (GGML Universal Format)
- **API**: `llama_adapter_lora_init(model, path_lora)` expects `.gguf` file
- **Metadata**: GGUF metadata format (not JSON config)

### Mismatch

```
Training → safetensors → ❌ llama.cpp can't load
Required → GGUF → ✅ llama.cpp can load
```

---

## Potential Solutions

### Option 1: Convert safetensors → GGUF (Recommended)

**Tool**: `convert-lora-to-gguf.py` from llama.cpp repository

**Steps**:

1. Train LoRA in safetensors format (current pipeline)
2. Convert to GGUF using `convert-lora-to-gguf.py`
3. Upload GGUF adapters to R2
4. Load in llama.cpp

**Pros**:

- ✅ Keep existing training pipeline
- ✅ Standard conversion tool
- ✅ Minimal code changes

**Cons**:

- ⚠️ Need to verify conversion script exists and works
- ⚠️ Additional conversion step in pipeline
- ⚠️ Need to handle both formats during development

**Implementation**:

```python
# Add to training pipeline
python llama.cpp/convert-lora-to-gguf.py \
    --lora-dir training/alice-3b-lora \
    --base-model microsoft/Phi-3-mini-4k-instruct \
    --output training/alice-3b-lora.gguf
```

### Option 2: Merge LoRA into Base Model (Not Ideal)

**Approach**: Merge LoRA weights into base model before quantization

**Steps**:

1. Train LoRA in safetensors
2. Merge LoRA into base model (using PEFT `merge_and_unload()`)
3. Export merged model to GGUF
4. Use merged model (no dynamic adapter loading)

**Pros**:

- ✅ No format conversion needed
- ✅ Works with existing pipeline

**Cons**:

- ❌ **Loses EVOLoRA Mesh architecture** (can't switch adapters dynamically)
- ❌ Need separate merged models for each adapter combination
- ❌ Defeats the purpose of multi-adapter system

**Why this doesn't work**: EVOLoRA Mesh requires **dynamic adapter switching** at inference time. Merging loses this capability.

### Option 3: Use Alternative Inference Engine

**Approaches**:

- **ONNX Runtime**: Convert to ONNX, but LoRA support is limited
- **CoreML**: Apple's framework, but LoRA support unclear
- **MLX**: Apple Silicon optimized, but iOS deployment complex
- **Transformers.js**: Web-based, but not native iOS

**Pros**:

- ✅ Might support safetensors natively
- ✅ Alternative optimization paths

**Cons**:

- ❌ Lose llama.cpp optimizations (Metal, memory efficiency)
- ❌ Major architecture change
- ❌ iOS deployment complexity
- ❌ Performance may be worse

### Option 4: Server-Side Adapter Merging

**Approach**: Merge adapters server-side, download merged models

**Steps**:

1. Train LoRA in safetensors
2. Server merges adapters based on Mesh decision
3. Server exports merged model to GGUF
4. Client downloads pre-merged GGUF model

**Pros**:

- ✅ Client only needs GGUF
- ✅ Server handles conversion

**Cons**:

- ❌ **Loses dynamic switching** (need to download new model for each adapter combination)
- ❌ Network overhead (downloading models per request)
- ❌ Latency (can't switch adapters mid-conversation)
- ❌ Defeats EVOLoRA Mesh architecture

---

## Recommended Solution: Option 1 (Conversion)

### Implementation Plan

#### Step 1: Verify Conversion Tool

```bash
# Check if convert-lora-to-gguf.py exists in llama.cpp
ls llama.cpp/convert-lora-to-gguf.py
# Or check if it's in tools/
ls llama.cpp/tools/convert-lora-to-gguf.py
```

#### Step 2: Create Conversion Script

```python
# scripts/convert_lora_to_gguf.py
import subprocess
import sys
from pathlib import Path

def convert_lora_to_gguf(
    lora_dir: str,
    base_model: str,
    output_path: str
):
    """Convert PEFT LoRA adapter to GGUF format."""
    # Find convert-lora-to-gguf.py in llama.cpp
    llama_cpp_dir = Path("llama.cpp")
    converter = llama_cpp_dir / "convert-lora-to-gguf.py"

    if not converter.exists():
        # Try tools directory
        converter = llama_cpp_dir / "tools" / "convert-lora-to-gguf.py"

    if not converter.exists():
        raise FileNotFoundError(
            "convert-lora-to-gguf.py not found. "
            "Clone llama.cpp repository first."
        )

    cmd = [
        sys.executable,
        str(converter),
        "--lora-dir", lora_dir,
        "--base-model", base_model,
        "--output", output_path,
    ]

    subprocess.run(cmd, check=True)
    print(f"✅ Converted LoRA to GGUF: {output_path}")
```

#### Step 3: Update Training Pipeline

```python
# training/train_phi3_alice.py (add at end)
if __name__ == "__main__":
    # ... existing training code ...

    # After training completes:
    print("\n📦 Converting LoRA adapter to GGUF...")
    convert_lora_to_gguf(
        lora_dir=OUTPUT_DIR,
        base_model=MODEL_ID,
        output_path=f"{OUTPUT_DIR}.gguf"
    )

    print(f"\n✅ LoRA adapter ready: {OUTPUT_DIR}.gguf")
    print("   Upload to R2: scripts/upload-gguf-to-r2.mjs")
```

#### Step 4: Update Asset Download

- Extend `AliceAssetDownloadManager` to download `.gguf` adapters
- Update R2 storage paths to use `.gguf` extension

---

## Verification Steps

1. **Check llama.cpp version**: Ensure it supports LoRA adapters

   ```bash
   # Check llama.cpp commit/version
   cd llama.cpp
   git log --oneline | head -5
   ```

2. **Test conversion**: Convert existing LoRA adapter

   ```bash
   python scripts/convert_lora_to_gguf.py \
       --lora-dir training/alice-3b-lora \
       --base-model Qwen/Qwen2.5-3B \
       --output training/alice-3b-lora.gguf
   ```

3. **Test loading**: Verify llama.cpp can load converted adapter
   ```swift
   // In LlamaEngine.swift
   let adapter = llama_adapter_lora_init(model, "training/alice-3b-lora.gguf")
   assert(adapter != nil, "Failed to load GGUF LoRA adapter")
   ```

---

## Alternative: Check llama.cpp LoRA Support Status

**Question**: Does the current llama.cpp version in `llama.xcframework` support LoRA adapters?

**Check**:

1. Verify API exists: ✅ (we saw `llama_adapter_lora_init` in headers)
2. Verify format: ❓ (need to confirm GGUF requirement)
3. Test with sample adapter: ❓ (need to test)

**If GGUF conversion doesn't work**, we may need to:

- Update llama.cpp to newer version with better LoRA support
- Or use alternative approach (merge adapters, different inference engine)

---

## Next Steps

1. **Immediate**: Verify `convert-lora-to-gguf.py` exists and works
2. **If conversion works**: Add to training pipeline
3. **If conversion doesn't work**: Evaluate alternative solutions
4. **Fallback**: Consider merging adapters (loses dynamic switching) or alternative inference engine

---

## References

- llama.cpp LoRA API: `flutter_app/ios/llama.xcframework/.../llama.h:624`
- Training output: `training/alice-3b-lora/adapter_model.safetensors`
- Compliance report: `docs/audits/EVOLoRA_Mesh_Compliance_Report.md`

## Related
