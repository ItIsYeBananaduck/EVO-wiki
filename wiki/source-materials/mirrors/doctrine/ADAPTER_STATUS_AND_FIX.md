---
title: ADAPTER_STATUS_AND_FIX
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/ADAPTER_STATUS_AND_FIX.md"]
updated: 2026-07-24
---

# Adapter Status and Fix Instructions

## Current Status

### ✅ META Adapter

- **Source**: `flutter_app/assets/action_lora_adapter/meta_lora.gguf` (49MB)
- **Architecture**: Fixed to `llama` ✅
- **In pubspec.yaml**: Yes ✅
- **Status**: Ready, but app needs rebuild to include in bundle

### ⚠️ ENF Adapter

- **Source**: `training/enf_lora/output/gguf/enforcer_lora.gguf` (corrupted from binary replace)
- **R2 Upload**: Uploaded, but may still have old architecture
- **Runtime**: Fixed in simulator location ✅ (temporary)
- **Issue**: Gets re-downloaded from R2 with wrong architecture during sync

### ⚠️ VOICE Adapter

- **Source**: `training/enf_lora/output/gguf/voice_lora.gguf` (may be corrupted)
- **R2 Upload**: Uploaded
- **Status**: Unknown - needs verification

## Immediate Fixes Applied

1. ✅ Fixed ENF adapter in runtime location (simulator)
2. ✅ Fixed VOICE adapter in runtime location (if it exists)
3. ✅ META adapter fixed in assets

## Next Steps

### 1. Rebuild App (Required for META Adapter)

```bash
cd flutter_app
flutter clean
flutter pub get
# Then rebuild in Xcode
```

After rebuild, the META adapter should be found at:

- Bundle: `action_lora_adapter/meta_lora.gguf`

### 2. Fix ENF Adapter Source and Re-upload

The source ENF adapter was corrupted by the binary replace. You need to:

**Option A: Re-export from original LoRA**

```bash
# If you have the original safetensors LoRA, re-convert to GGUF
# with correct architecture metadata
```

**Option B: Fix the corrupted source file**

```bash
# Download from R2, fix it, then re-upload
cd /Users/user287043/Documents/git-fit
python3 training/enf_lora/scripts/fix_architecture_simple.py \
    training/enf_lora/output/gguf/enforcer_lora.gguf
```

Then re-upload:

```bash
npx wrangler r2 object put alice-assets/adapters/enforcer/enforcer_lora.gguf \
    --file=training/enf_lora/output/gguf/enforcer_lora.gguf \
    --content-type=application/octet-stream
```

### 3. Delete Cached Adapters (After Re-upload)

After fixing and re-uploading to R2, delete the cached adapters so they re-download:

```bash
# Delete in simulator (adjust device ID as needed)
rm "/Users/user287043/Library/Developer/CoreSimulator/Devices/21176330-198B-41D3-90C4-315DB1099A05/data/Containers/Shared/AppGroup/3B48DC78-4F04-4864-9419-533BBF8C6056/EVO/ModelStore/AliceAssets/adapters/enforcer/enforcer_lora.gguf"
rm "/Users/user287043/Library/Developer/CoreSimulator/Devices/21176330-198B-41D3-90C4-315DB1099A05/data/Containers/Shared/AppGroup/3B48DC78-4F04-4864-9419-533BBF8C6056/EVO/ModelStore/AliceAssets/adapters/voice/voice_lora.gguf"
```

Then restart the app to trigger re-download.

## Verification

After fixes, check logs for:

- ✅ `[LlamaEngine] META adapter prepended to stack` (not "not found")
- ✅ `[llama.cpp] llama_adapter_lora_init_impl: - kv   0: general.architecture str = llama` (not "phi3")
- ✅ No "model arch and LoRA arch mismatch" errors

## Root Cause

The binary replace script (`fix_architecture_simple.py`) works for fixing runtime files, but may corrupt the source GGUF files if the metadata structure is complex. For source files, it's better to:

1. Re-export from original LoRA with correct architecture
2. Or use a proper GGUF metadata editor that preserves file structure

## Related

^[{src_rel}]
