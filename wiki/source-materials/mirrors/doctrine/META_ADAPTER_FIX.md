---
title: META_ADAPTER_FIX
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/META_ADAPTER_FIX.md"]
updated: 2026-07-24
---

# Fix: META Adapter Not Found

The retrained `meta_lora.gguf` adapter is not being loaded. Here's how to fix it:

## Problem

The app is looking for the META adapter at:

1. `AliceAssets/adapters/meta/meta_lora.gguf` (runtime location)
2. `action_lora_adapter/meta_lora.gguf` (in app bundle)

But it's not finding it, so you see:

```
[LlamaEngine] WARNING: META adapter not found - Alice cannot self-learn without it
```

## Solution

### Option 1: Include in Flutter Assets (Recommended)

Add the adapter to `pubspec.yaml` so it's bundled with the app:

1. **Edit `flutter_app/pubspec.yaml`** and add to the `flutter:` section:

```yaml
flutter:
  assets:
    - assets/action_lora_adapter/meta_lora.gguf
```

2. **Run:**

```bash
cd flutter_app
flutter pub get
flutter clean
flutter build ios
```

### Option 2: Copy to Runtime Location

Copy the adapter to where the app expects it at runtime:

```bash
# The app looks in App Group or Documents/EVO/ModelStore/AliceAssets/adapters/meta/
# You'll need to copy it there after the app runs, or include it in the asset sync
```

### Option 3: Verify Bundle Inclusion

Check if the file is already in the bundle but not being found:

1. Build the app
2. Check the app bundle:

```bash
# After building, check:
unzip -l build/ios/iphoneos/Runner.app | grep meta_lora
```

## Verify It's Working

After fixing, you should see in logs:

```
[LlamaEngine] META adapter prepended to stack (path: ..., scale: 1.0)
```

Instead of:

```
[LlamaEngine] WARNING: META adapter not found
```

## Additional Issue: ENF Adapter Architecture Mismatch

The ENF adapter is also failing with:

```
llama_adapter_lora_init: failed to apply lora adapter: model arch and LoRA arch mismatch
```

This is because:

- Base model architecture: `llama` (even though it's called "phi3")
- ENF adapter architecture: `phi3`

The ENF adapter needs to be retrained/converted for the `llama` architecture, or the base model needs to be a true Phi-3 model.

For now, the META adapter fix is the priority since that's what you just retrained.

## Related
