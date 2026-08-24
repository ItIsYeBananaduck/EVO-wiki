---
title: GGUF_CONVERSION_GUIDE
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/GGUF_CONVERSION_GUIDE.md"]
updated: 2026-07-24
---

# GGUF Conversion Guide: ENF and VOICE LoRAs

## Overview

After training ENF and VOICE LoRAs, they must be converted to GGUF format to work with llama.cpp on iOS/Android.

## Training Output Format

LoRA training typically outputs adapters in one of these formats:

- **PyTorch format**: `.safetensors` or `.bin` (standard LoRA checkpoint)
- **MLX format**: `.safetensors` (for MLX training)
- **llama.cpp format**: Direct GGUF (if using llama.cpp-based training)

## Conversion Tools

### Option 1: Using llama.cpp's convert-lora-to-gguf (Recommended)

This is the official tool for converting LoRA adapters to GGUF format.

#### Installation

1. **Clone llama.cpp** (if not already available):

   ```bash
   git clone https://github.com/ggerganov/llama.cpp.git
   cd llama.cpp
   make convert-lora-to-gguf
   ```

2. **Or install via pip** (if available):
   ```bash
   pip install llama-cpp-python[gguf] --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cpu
   ```

#### Conversion Process

**For ENF LoRA:**

```bash
# Assuming base model is already in GGUF format
python llama.cpp/convert-lora-to-gguf.py \
  --base-model alice-phi3-q4.gguf \
  --lora enforcer_lora.safetensors \
  --output enforcer_lora.gguf \
  --out-type f16  # or q4_k_m for quantization
```

**For VOICE LoRA:**

```bash
python llama.cpp/convert-lora-to-gguf.py \
  --base-model alice-phi3-q4.gguf \
  --lora voice_lora.safetensors \
  --output voice_lora.gguf \
  --out-type f16  # or q4_k_m for quantization
```

### Option 2: Using llama.cpp's merge-adapters (If training produces GGUF directly)

If your training pipeline already produces GGUF adapters, you may still need to merge with base:

```bash
python llama.cpp/scripts/merge-adapters.py \
  --base-model alice-phi3-q4.gguf \
  --adapter enforcer_lora.gguf \
  --output enforcer_lora_merged.gguf
```

**Note**: For LoRA adapters (not full fine-tuned models), you typically keep them separate and use `llama_adapter_lora_init()` at runtime.

### Option 3: Using litellm/llama-factory conversion (If available)

Some training frameworks include conversion tools:

```bash
# Example with llama-factory (if using it)
python llama_factory/export.py \
  --model_name_or_path ./checkpoints/enf_lora \
  --adapter_name_or_path ./checkpoints/enf_lora \
  --template phi3 \
  --finetuning_type lora \
  --export_dir ./exports \
  --export_size 2 \
  --export_legacy_format false
```

## Important Notes

### 1. Base Model Compatibility

- **ENF and VOICE must use the same base model** as your app: `alice-phi3-q4.gguf`
- The base model hash/version must match exactly
- Conversion tools should validate base model compatibility

### 2. LoRA Rank and Alpha

- **ENF**: Rank 8 or 16, Alpha 16 or 32 (as specified in training)
- **VOICE**: Rank 8, Alpha 16 (as specified in training)
- These parameters are embedded in the GGUF file metadata

### 3. Quantization (Optional)

LoRA adapters are typically kept in F16 (float16) for best quality:

- **F16**: Higher quality, ~10-30MB per adapter (rank 8-16)
- **Q4_K_M**: Quantized, smaller size (~5-15MB), may reduce effectiveness

**Size Reference for Phi-3 8B**:

- Rank 8 LoRA: ~10-20MB in GGUF F16 format
- Rank 16 LoRA: ~20-30MB in GGUF F16 format
- Base model (Q4_K_M): 2.2GB - LoRA adapters are **1-2%** of base model size

For ENF (safety-critical), **recommend F16** (no quantization).
For VOICE (style), quantization may be acceptable if size is a concern.

### 4. File Verification

After conversion, verify the GGUF file:

```bash
# Check GGUF magic bytes
xxd -l 4 enforcer_lora.gguf
# Should show: 47 47 55 46 ("GGUF")

# Check file size (should match expectations)
ls -lh enforcer_lora.gguf voice_lora.gguf

# Test loading (if you have llama.cpp tools)
./llama.cpp/main -m alice-phi3-q4.gguf --lora enforcer_lora.gguf --n-predict 1
```

## Conversion Checklist

Before uploading to R2:

- [ ] ENF LoRA converted to GGUF format
- [ ] VOICE LoRA converted to GGUF format
- [ ] Both use same base model (alice-phi3-q4.gguf)
- [ ] GGUF magic bytes verified (first 4 bytes = "GGUF")
- [ ] File sizes reasonable (50-100MB for F16, 25-50MB for Q4_K_M)
- [ ] Tested loading with llama.cpp (optional but recommended)
- [ ] Metadata JSON created (version, checksum, etc.)

## Post-Conversion Steps

1. **Update expected file sizes** in `alice_asset_download_manager.dart`:

   ```dart
   // ENF LoRA
   expectedSizeBytes: <actual_size_in_bytes>,

   // VOICE LoRA
   expectedSizeBytes: <actual_size_in_bytes>,
   ```

2. **Compute checksums** for metadata:

   ```bash
   shasum -a 256 enforcer_lora.gguf > enforcer_lora.gguf.sha256
   shasum -a 256 voice_lora.gguf > voice_lora.gguf.sha256
   ```

3. **Create metadata JSON files** (optional but recommended):

   ```json
   {
     "version": "20250101.1",
     "checksum": "sha256:...",
     "trainedOnGuardrailVersion": "1.0",
     "baseModelId": "alice-phi3-q4",
     "rank": 8,
     "alpha": 16,
     "steps": 2000,
     "format": "gguf",
     "quantization": "f16"
   }
   ```

4. **Upload to R2** following `R2_UPLOAD_INSTRUCTIONS_ENF_VOICE.md`

## Troubleshooting

### "Base model mismatch" error

- Ensure you're using the exact same base model file that the app uses
- Check base model hash/version

### "Invalid GGUF format" error

- Verify magic bytes are correct
- Check if conversion tool version is compatible with llama.cpp version in app

### Adapter not loading at runtime

- Verify adapter path matches `getEnforcerAdapterPath()` / `getVoiceAdapterPath()` in iOS
- Check file permissions (should be readable)
- Verify adapter scale values (ENF: 0.9-1.0, VOICE: 0.7-0.8)

### Large file sizes

- Consider quantization if needed (Q4_K_M)
- Verify rank/alpha aren't too high (causing bloated adapters)

## References

- llama.cpp LoRA support: https://github.com/ggerganov/llama.cpp/blob/master/examples/llama-lora/README.md
- GGUF specification: https://github.com/ggerganov/ggml/blob/master/docs/gguf.md
- LoRA training: See training scripts in `training/enf_lora/scripts/`

## Related
