---
title: CRASH_FIX_162_SIGABRT
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CRASH_FIX_162_SIGABRT.md"]
updated: 2026-07-24
---

# Crash Fix: SIGABRT in llama.cpp (Build 162)

## Issue

The app was crashing with `EXC_CRASH (SIGABRT)` in Thread 3, triggered by llama.cpp assertion failures. The crash occurred during model inference when batch positions exceeded the context window bounds.

## Root Cause

The crash was caused by insufficient validation of position values (`nPast` and batch positions) before calling `llama_decode()`. llama.cpp has internal assertions that call `abort()` when:

1. Batch positions exceed the context window size (`n_ctx`)
2. Batch size exceeds the configured batch size (`n_batch`)
3. Position values are negative or invalid

The code was incrementing `nPast` after each chunk without validating that the new value wouldn't exceed bounds, and there were edge cases where invalid configurations could slip through.

**Additional Finding**: The crash occurs very early (within 5-6 seconds of app launch), indicating that inference is being triggered immediately after model load. The app calls `loadModel()` in `initState()` of the chat screen, and if there's an initial message, it gets sent as soon as the model is marked as ready. This can cause inference to start before the model context is fully stable.

## Fixes Applied

### 1. Added effectiveNctx Validation

- Validate `effectiveNctx > 0` before any position calculations
- Prevents crashes from invalid/zero context size configurations

### 2. Added effectiveBatch Validation

- Validate `effectiveBatch > 0` before batch size checks
- Ensures batch size limits are valid

### 3. Added nPast Overflow Protection

- Validate `nPast` won't exceed `effectiveNctx` after incrementing
- Added check: `newNPast < effectiveNctx` before updating `nPast`
- Prevents position overflow that would trigger SIGABRT

### 4. Enhanced Position Validation

- Validate all batch positions are within bounds `[0, effectiveNctx)`
- Added validation before setting positions in batch arrays
- Added final validation right before `llama_decode()` call

### 5. Added Context Validation

- Validate context pointer is not nil before decode
- Double-check batch structure integrity before decode

### 6. Applied Same Protections to VOICE Generation

- Added same validations to `generateVoiceStyledAnswer()` function
- Ensures VOICE pass is also protected from position overflow

### 7. Added Early Inference Prevention

- Added `effectiveConfig` validation at the start of `generate()` function
- Prevents inference from starting if configuration is invalid
- Added final validation after model load to ensure context is stable
- Added context/model validation after Metal warmup

## Code Changes

### Main Generation Loop (`generate()`)

- Added `effectiveNctx > 0` validation at start
- Added `nPast` overflow check after increment (line ~3998)
- Enhanced position validation in batch building (line ~3763)
- Added final validation before decode (line ~2013)

### VOICE Generation (`generateVoiceStyledAnswer()`)

- Added `effectiveNctx > 0` validation
- Added position validation in batch building
- Added `nPast` overflow check after increment

### Safe Decode Function (`safeDecode()`)

- Added `effectiveNctx > 0` validation
- Added `effectiveBatch > 0` validation
- Added final position validation before `llama_decode()` call

## Testing Recommendations

1. **Test with large prompts** that approach context window limits
2. **Test with edge cases**:
   - Prompts that exactly fill the context window
   - Prompts that would exceed context window (should fail gracefully)
   - Rapid successive generation requests
3. **Monitor crash logs** for any remaining SIGABRT crashes
4. **Verify error messages** are logged correctly when bounds are exceeded

## Prevention

The fixes add multiple layers of validation:

1. **Early validation**: Check config validity before starting
2. **Per-chunk validation**: Validate positions before building each batch
3. **Pre-increment validation**: Check bounds before incrementing `nPast`
4. **Pre-decode validation**: Final check right before calling `llama_decode()`

This defense-in-depth approach ensures that even if one validation is missed, others will catch the issue before it causes a crash.

## Related Files

- `flutter_app/ios/Runner/LlamaEngine.swift` - Main fixes applied here
- `flutter_app/ios/Runner/CrashLogger.swift` - Already has SIGABRT signal handler

## Status

✅ Fixed - All critical validations added to prevent SIGABRT crashes from position overflow
