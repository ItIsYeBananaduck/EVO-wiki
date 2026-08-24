---
title: CRASH_FIX_SUMMARY
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CRASH_FIX_SUMMARY.md"]
updated: 2026-07-24
---

# LlamaEngine Crash Fix Summary

## Issue

SIGABRT crashes occurring in `llama_decode` calls within `LlamaEngine.safeDecode` function, causing app crashes in TestFlight builds.

## Root Cause Analysis

The crashes were happening inside the llama.cpp library's `llama_decode` function, which calls `abort()` when it detects invalid state. Despite validation, crashes persisted, indicating:

1. Context invalidation between validation and decode
2. Batch structure corruption
3. Memory alignment or pointer issues
4. Potential race conditions despite locking

## Comprehensive Fixes Applied

### 1. Context Validation

- Added checks to ensure context pointer matches `self.context` before decode
- Multiple validation points to catch context invalidation
- Final check immediately before `llama_decode` call

### 2. Batch Structure Validation

- **Batch size validation**: Ensures `batch.n_tokens` is within bounds (1 to n_batch)
- **Pointer validation**: Verifies `batch.token`, `batch.pos`, and `batch.logits` pointers are not nil
- **Position bounds checking**: Validates all positions are within `[0, n_ctx)` range
- **Token ID validation**: Ensures all token IDs are non-negative
- **Seq ID validation**: Properly handles optional `seq_id` pointer arrays
- **Batch integrity check**: Verifies batch structure is consistent before decode

### 3. Batch Setup Validation

- Added validation when reusing batches during generation
- Ensures batch pointers are valid before reuse
- Validates position is within bounds before setting
- Verifies batch was set correctly after initialization

### 4. Diagnostic Logging

- Added comprehensive logging before decode calls
- Logs batch state (n_tokens, positions, logits) for crash diagnosis
- Forces log flush before potentially fatal operations
- Captures llama.cpp internal logs on failures

### 5. Error Recovery

- Returns `shouldRetryWithCPU` flag for Metal failures
- Graceful degradation when decode fails
- Proper error propagation to callers

## Files Modified

1. **flutter_app/ios/Runner/LlamaEngine.swift**
   - Enhanced `safeDecode` function with comprehensive validation
   - Added batch setup validation in generation loop
   - Improved error handling and logging

## Testing Recommendations

1. **Monitor crash logs** for the new diagnostic messages
2. **Check batch state logs** to identify patterns in crashes
3. **Verify context lifecycle** to ensure no premature freeing
4. **Test with different batch sizes** to identify edge cases
5. **Monitor Metal vs CPU performance** to detect fallback triggers

## Next Steps if Crashes Persist

1. **Check llama.cpp version**: Update to latest version if using older build
2. **Review llama.cpp GitHub issues**: Search for similar crash reports
3. **Consider batch allocation**: Ensure batches are properly allocated with sufficient capacity
4. **Thread safety audit**: Verify all llama.cpp calls are properly locked
5. **Memory profiling**: Check for memory corruption or leaks

## Key Validation Points

The `safeDecode` function now validates:

- ✅ Context is not nil
- ✅ Context matches `self.context`
- ✅ Model is not nil
- ✅ InferenceConfig is valid
- ✅ Batch has tokens (n_tokens > 0)
- ✅ Batch size doesn't exceed n_batch
- ✅ All positions are within [0, n_ctx)
- ✅ All token IDs are non-negative
- ✅ Seq ID pointers are valid (if present)
- ✅ Batch memory pointers are not nil
- ✅ Batch structure integrity
- ✅ First token/position are accessible and valid

## Crash Prevention Strategy

1. **Defense in depth**: Multiple validation layers
2. **Fail fast**: Return early with detailed error messages
3. **Diagnostic logging**: Capture state before potential crashes
4. **Graceful degradation**: CPU fallback for Metal failures
5. **Comprehensive error handling**: Proper error propagation

## Status

✅ All defensive checks implemented
✅ Compilation verified
✅ No linter errors
⏳ Awaiting TestFlight validation

## Related
