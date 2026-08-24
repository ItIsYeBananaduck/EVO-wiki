---
title: ADAPTER_FIX_SUMMARY
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/ADAPTER_FIX_SUMMARY.md"]
updated: 2026-07-24
---

# Adapter Architecture Fix Summary

## Problem

Both adapters had `general.architecture = phi3` but the base model reports `general.architecture = llama`, causing `llama.cpp` to reject them with "model arch and LoRA arch mismatch".

## Root Cause

- The base model (Phi-3) is stored in GGUF format which reports architecture as "llama" (this is correct for GGUF)
- The adapters were created with architecture metadata set to "phi3"
- `llama.cpp` requires exact architecture match between base model and adapters

## Fix Applied

1. **META adapter**: Fixed in `flutter_app/assets/action_lora_adapter/meta_lora.gguf` (source)
2. **ENF adapter**: Fixed in runtime location (but gets overwritten by server sync)

## Current Issues

### 1. ENF Adapter Gets Overwritten

The ENF adapter is downloaded from the server (R2/Supabase) during app sync. The server still has the old `phi3` version, so every sync overwrites our local fix.

**Solution**: Upload the fixed ENF adapter to the server:

```bash
# Fix the source file first
python3 training/enf_lora/scripts/fix_architecture_simple.py \
    flutter_app/assets/action_lora_adapter/enforcer_lora.gguf

# Then upload to R2/Supabase (check upload scripts)
```

### 2. META Adapter Not Found

The META adapter is in `pubspec.yaml` but the app hasn't been rebuilt, so it's not in the bundle.

**Solution**: Rebuild the app:

```bash
cd flutter_app
flutter clean
flutter pub get
flutter build ios  # or just run it
```

## Temporary Workaround

For testing, you can fix the ENF adapter in the runtime location after each app launch:

```bash
python3 training/enf_lora/scripts/fix_architecture_simple.py \
    "/path/to/simulator/EVO/ModelStore/AliceAssets/adapters/enforcer/enforcer_lora.gguf"
```

But this will be overwritten on the next sync.

## Permanent Fix

1. Fix the source ENF adapter file
2. Upload fixed ENF adapter to server (R2/Supabase)
3. Rebuild app to include META adapter in bundle
4. Both adapters will then work correctly

## Related
