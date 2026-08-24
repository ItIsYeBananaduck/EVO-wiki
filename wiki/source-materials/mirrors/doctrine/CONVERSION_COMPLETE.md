---
title: CONVERSION_COMPLETE
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CONVERSION_COMPLETE.md"]
updated: 2026-07-24
---

# LoRA Conversion Complete ✅

**Date**: January 14, 2025

## Summary

Successfully converted all downloaded VOICE LoRA adapters from safetensors to GGUF format.

## Converted Files

### VOICE LoRA Adapters (4 checkpoints)

All checkpoints converted to GGUF format:

| Checkpoint      | Source                                                    | Output                                       | Size    | Status |
| --------------- | --------------------------------------------------------- | -------------------------------------------- | ------- | ------ |
| checkpoint-250  | `output/downloads/voice_lora/voice_lora/checkpoint-250/`  | `output/gguf/voice_lora_checkpoint_250.gguf` | ~3.0 MB | ✅     |
| checkpoint-500  | `output/downloads/voice_lora/voice_lora/checkpoint-500/`  | `output/gguf/voice_lora_checkpoint_500.gguf` | ~3.0 MB | ✅     |
| checkpoint-750  | `output/downloads/voice_lora/voice_lora/checkpoint-750/`  | `output/gguf/voice_lora_checkpoint_750.gguf` | ~3.0 MB | ✅     |
| checkpoint-1000 | `output/downloads/voice_lora/voice_lora/checkpoint-1000/` | `output/gguf/voice_lora.gguf`                | ~3.0 MB | ✅     |

**Recommended**: Use `voice_lora.gguf` (checkpoint-1000) as it's the most trained.

### ENF LoRA Adapters

**Status**: ⚠️ No safetensors files found in ENF checkpoints

ENF checkpoints contain:

- `adapter_config.json` ✅
- Training state files (rng_state.pth, scheduler.pt, etc.)
- Tokenizer files
- **Missing**: `adapter_model.safetensors` or `adapter_model.bin`

**Note**: ENF adapters may need to be extracted from training state or converted differently.

## File Verification

All GGUF files verified:

- ✅ Magic bytes: `GGUF` (0x47475546)
- ✅ File sizes: ~3.0 MB each (reasonable for rank 8 LoRA)
- ✅ Checksums computed: `output/gguf/checksums.sha256`

## Conversion Details

- **Base Model**: `training/alice-phi3-mlx/alice-phi3-q4.gguf` (2.2 GB)
- **Output Format**: F16 (float16) - highest quality
- **Conversion Tool**: `alicecore_build/llama.cpp/convert_lora_to_gguf.py`

## Next Steps

### 1. Test the Adapters (Optional)

```bash
# If you have llama.cpp built
./alicecore_build/llama.cpp/build/bin/llama-cli \
  -m training/alice-phi3-mlx/alice-phi3-q4.gguf \
  --lora output/gguf/voice_lora.gguf \
  -p "Hello" -n 10
```

### 2. Upload to R2 Storage

See `TRAINING_TO_R2_GUIDE.md` for upload instructions:

```bash
# Upload VOICE LoRA
wrangler r2 object put alice-assets/adapters/voice/voice_lora.gguf \
  --file=./output/gguf/voice_lora.gguf
```

### 3. Update App Code

Update expected file sizes in `alice_asset_download_manager.dart`:

- VOICE LoRA: ~3.0 MB (3,145,728 bytes)

### 4. Handle ENF LoRA

ENF adapters need investigation:

- Check if adapters are embedded in training state
- May need to extract from PyTorch checkpoints
- Or re-train with explicit adapter export

## File Locations

```
training/enf_lora/
├── output/
│   ├── downloads/          # Original safetensors from Drive
│   │   ├── voice_lora/
│   │   └── enf_checkpoints/
│   └── gguf/               # Converted GGUF files ✅
│       ├── voice_lora.gguf (checkpoint-1000)
│       ├── voice_lora_checkpoint_250.gguf
│       ├── voice_lora_checkpoint_500.gguf
│       ├── voice_lora_checkpoint_750.gguf
│       └── checksums.sha256
```

## Checksums

See `output/gguf/checksums.sha256` for SHA256 checksums of all converted files.

## Notes

- All VOICE LoRA adapters successfully converted
- ENF LoRA adapters require additional investigation
- Files are ready for R2 upload and app integration
- Using F16 format for best quality (no quantization)

## Related
