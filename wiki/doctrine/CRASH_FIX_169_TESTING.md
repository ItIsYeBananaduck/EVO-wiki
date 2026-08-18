---
title: CRASH_FIX_169_TESTING
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CRASH_FIX_169_TESTING.md"]
updated: 2026-07-24
---

# Crash Fix 169 - Testing Guide

## Summary

Crash persists at `llama_decode` despite batch reuse fixes. This suggests either:

1. Metal-specific bug in llama.cpp
2. Memory corruption we can't detect from Swift
3. Threading issue in llama.cpp itself

## Testing Steps

### 1. Test on Simulator (CPU-only)

The simulator already disables Metal automatically. Run the app on simulator to see if crash still occurs.

### 2. Force CPU Mode on Device

To test if crash is Metal-specific on device:

**Option A: Environment Variable**

```bash
# Set before running app
export LLAMA_FORCE_CPU=1
```

**Option B: UserDefaults (in app)**

```swift
UserDefaults.standard.set(true, forKey: "llama_force_cpu")
```

This will force CPU mode even when GPU layers are requested.

### 3. Check llama.cpp Version

Current version: **0.0.188** (from `llama-version.cmake`)

Check if there are known issues with this version:

- Search llama.cpp GitHub issues for "SIGABRT" or "llama_decode crash"
- Check if newer versions have fixes

## Changes Made

### Batch Management

1. **Batch reuse eliminated**: All batches created fresh each use (no more `n_tokens = 0` resets)
2. **Thread safety**: All batch creation moved INSIDE locks to prevent race conditions
3. **Comprehensive validation**: All batch fields validated before `llama_decode` call

### Locations Fixed

- Main generation batch (line ~4916)
- Prompt batch (line ~4261)
- Repair function batches (lines ~8344, ~8415)
- VOICE prompt batch (line ~8865)
- VOICE generation batch (line ~9026)

### CPU Mode Override

- Added `LLAMA_FORCE_CPU` environment variable support
- Added `llama_force_cpu` UserDefaults key support
- Allows testing CPU mode on device to isolate Metal issues

## Next Steps if Crash Persists

1. **Test on simulator**: If crash doesn't happen, it's Metal-specific
2. **Test with CPU mode**: Use `LLAMA_FORCE_CPU=1` to force CPU on device
3. **File issue with llama.cpp**: If crash happens in CPU mode too, it's likely a llama.cpp bug
4. **Check llama.cpp version**: Consider updating to latest version
5. **Review llama.cpp issues**: Search for similar crash reports

## Known Issues

- llama.cpp version 0.0.188 may have bugs
- Metal acceleration may have threading issues
- Batch structure validation may not catch all corruption cases
- llama.cpp calls `abort()` internally which we can't prevent from Swift

## Related

^[{src_rel}]
