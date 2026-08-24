---
title: FIX_ENF_ARCHITECTURE
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/FIX_ENF_ARCHITECTURE.md
updated: 2026-07-24
---

# Fix ENF Adapter Architecture Mismatch

## Problem

The ENF adapter fails to load with:

```
llama_adapter_lora_init: failed to apply lora adapter: model arch and LoRA arch mismatch
```

**Root Cause:**

- Base model (Phi-3) reports: `general.architecture = llama` in GGUF metadata
- ENF adapter reports: `general.architecture = phi3` in GGUF metadata
- Even though both are Phi-3 models, the architecture strings don't match

## Solution

Update the ENF adapter's architecture metadata to "llama" to match the base model.

### Option 1: Use gguf-py Script (Recommended)

```bash
cd training/enf_lora/scripts

# Install gguf-py if needed
pip install gguf

# Update the adapter metadata
python -m gguf.scripts.gguf_new_metadata \
    --input ../../../flutter_app/assets/action_lora_adapter/enforcer_lora.gguf \
    --output ../../../flutter_app/assets/action_lora_adapter/enforcer_lora.gguf \
    --general-architecture llama
```

### Option 2: Re-convert the Adapter

If you have the original safetensors adapter, re-convert it with the correct architecture:

```bash
# Use convert_lora_to_gguf.py with --base-model-id pointing to a model
# that reports "llama" architecture
python alicecore_build/llama.cpp/convert_lora_to_gguf.py \
    --base-model-id microsoft/Phi-3-mini-4k-instruct \
    --architecture llama \
    adapter_model.safetensors \
    enforcer_lora.gguf
```

### Option 3: Use llama.cpp Tools

If you have llama.cpp built:

```bash
# Use gguf-dump to check current metadata
./alicecore_build/llama.cpp/build/bin/gguf-dump \
    flutter_app/assets/action_lora_adapter/enforcer_lora.gguf | grep architecture

# Use gguf-set to update (if available)
# Or use Python script above
```

## Verify Fix

After updating, the adapter should load without the architecture mismatch error:

```
[LlamaEngine] ENF adapter prepended to stack (path: ..., scale: 1.0)
```

Instead of:

```
[LlamaEngine] ERROR: Failed to load LoRA adapter: model arch and LoRA arch mismatch
```

## Why This Happens

Phi-3 models are based on Llama architecture internally. When converted to GGUF:

- Some converters set `general.architecture = phi3`
- Others set `general.architecture = llama` (the underlying architecture)
- llama.cpp requires an exact match, so adapters must use the same string as the base model

## Note

This only affects the ENF adapter. The META adapter should work fine once it's included in the bundle (see META_ADAPTER_FIX.md).

## Related

^[source-materials/mirrors/doctrine/FIX_ENF_ARCHITECTURE.md]
