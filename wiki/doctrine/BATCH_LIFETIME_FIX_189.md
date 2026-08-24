---
title: BATCH_LIFETIME_FIX_189
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/BATCH_LIFETIME_FIX_189.md
updated: 2026-07-24
---

# Batch Lifetime Fix 189 - Implementation Summary

## Root Cause Analysis

**Crash Details:**

- Exception: `EXC_CRASH (SIGABRT)`
- Triggered by: Thread 6
- Crash location: `llama.framework` during `llama_decode`
- Call stack:
  - Frame 14: `closure #2 in LlamaEngine.safeDecode` (LlamaEngine.swift:3299)
  - Frame 28: `LlamaEngine.safeDecode` (LlamaEngine.swift:3296)
  - Frame 29: `LlamaEngine._processGeneration` (LlamaEngine.swift:5393)

**Root Cause:**
The crash is caused by **batch pointer lifetime bugs**:

1. Batches created with `llama_batch_init` and `defer { llama_batch_free(...) }`
2. `safeDecode` dispatches to `llamaQueue` (potentially async)
3. Function returns → `defer` runs → batch is freed
4. But `llama_decode` may still be running or hasn't started yet
5. `llama.cpp` accesses freed batch pointers → `SIGABRT`

**Evidence:**

- Crash occurs inside `llama_decode` after validation passes
- Stack shows dispatch queue activity (libswiftDispatch.dylib)
- Batch validation passes, but decode still crashes

## Implemented Fixes

### 1. Lifetime-Safe Batch Helpers ✅

Created two helper functions that ensure batch lifetime safety:

#### `decodeTokensChunk(...)` - For prompt chunks

- Creates batch, populates it, decodes, and frees it **all in one synchronous scope on llamaQueue**
- Batch cannot be freed until `llama_decode` completes
- Location: Lines ~2560-2800

#### `decodeSingleToken(...)` - For generation tokens

- Same pattern for single-token batches
- Location: Lines ~2800-2950

**Key Invariant:**

```swift
executeOnLlamaQueue {
    var batch = llama_batch_init(...)
    defer { llama_batch_free(batch) }  // Only freed after decode completes

    // Populate batch
    // ...

    // Decode (synchronous, in same scope)
    llama_decode(ctx, batch)

    // Batch freed here when scope exits
}
```

### 2. Updated All Batch Creation Sites ✅

**Prompt batches:**

- Line ~5217: Main prompt chunk processing → `decodeTokensChunk`
- Line ~9986: VOICE prompt chunks → `decodeTokensChunk`
- Line ~9460: Repair prompt → `decodeTokensChunk`

**Generation batches:**

- Line ~6049: Main generation loop → `decodeSingleToken`
- Line ~10147: VOICE generation → `decodeSingleToken`
- Line ~9527: Repair generation → `decodeSingleToken`

**Special cases:**

- Line ~2290: GPU smoke test → `decodeSingleToken`
- Line ~2393: Metal warmup → `decodeSingleToken`

### 3. Safety Asserts ✅

**Batch pointer validation:**

- `batch.token != nil`
- `batch.pos != nil`
- `batch.n_seq_id != nil`
- `batch.seq_id != nil`
- `batch.logits != nil`

**Bounds checking:**

- `idx < chunkSize` (capacity check)
- `pos >= 0 && pos < n_ctx`
- `token >= 0`
- `batch.n_tokens <= capacity`

All failures return error instead of crashing.

### 4. Fixed seq_id Assignment ✅

**Before (unsafe):**

```swift
promptBatch.n_seq_id[idx] = 1
if let seqId = promptBatch.seq_id[idx] {
    seqId[0] = 0
}
```

**After (deterministic):**

```swift
batch.n_seq_id[idx] = 1

// CRITICAL: seq_id[idx] must be non-nil when n_seq_id[idx] == 1
guard let seqIdPtr = batch.seq_id[idx] else {
    // FATAL: Log and return error
    return (false, false)
}
seqIdPtr[0] = 0
```

### 5. Re-Entrancy Guards ✅

Added `isDecoding` flag to prevent concurrent decode:

- `enterDecode()` checks `isDecoding` and returns `false` if decode already in progress
- `exitDecode()` clears `isDecoding`
- Both helpers check `enterDecode()` return value and fail gracefully

### 6. Lightweight Batch Logging ✅

Before every `llama_decode`, log:

- `operation`, `n_tokens`
- `firstToken`, `lastToken`
- `firstPos`, `lastPos`
- `tokenPtr`, `posPtr` (pointer addresses)

Logs are lightweight (in-memory only, no file I/O) and help diagnose crashes.

## Files Changed

1. **LlamaEngine.swift**:
   - Added `decodeTokensChunk()` helper (lines ~2560-2800)
   - Added `decodeSingleToken()` helper (lines ~2800-2950)
   - Updated `enterDecode()`/`exitDecode()` with re-entrancy guards (lines ~1058-1098)
   - Updated all batch creation sites to use helpers:
     - Prompt chunks: ~5217, ~9986, ~9460
     - Generation tokens: ~6049, ~10147, ~9527
     - Smoke test/warmup: ~2290, ~2393

## Test Plan

### 1. Long Prompt Near n_ctx

- **Test**: Send prompt with `nPast + promptTokens ≈ n_ctx - 10`
- **Expected**: Decode succeeds, batch lifetime is safe
- **Verify**: No SIGABRT, batch pointers remain valid throughout decode

### 2. Rapid Generate/Cancel

- **Test**: Start generation, immediately cancel, repeat 10x rapidly
- **Expected**: No crashes, all batches properly freed
- **Verify**: Re-entrancy guard prevents concurrent decode

### 3. Multiple Concurrent Requests

- **Test**: Send multiple generation requests simultaneously
- **Expected**: Requests are serialized, no concurrent decode
- **Verify**: `isDecoding` flag prevents overlap

### 4. Unload During Generation

- **Test**: Start generation, immediately call `unloadModel()`
- **Expected**: Unload waits for decode completion (max 10s)
- **Verify**: `waitForDecodeCompletion()` works correctly

### 5. Batch Pointer Validation

- **Test**: Force batch creation with invalid pointers (if possible)
- **Expected**: Early failure with error, no crash
- **Verify**: All pointer validations catch issues

## Key Improvements

1. **Batch Lifetime Safety**: Batches are created, used, and freed in the same synchronous scope
2. **No Queue Boundary Crossings**: Batch never crosses queue boundaries
3. **Re-Entrancy Protection**: Concurrent decode attempts are blocked
4. **Comprehensive Validation**: All batch pointers and bounds are validated
5. **Deterministic seq_id**: seq_id assignment is safe and validated
6. **Better Diagnostics**: Lightweight logging helps diagnose any remaining issues

## Expected Outcome

- **Eliminates SIGABRT crashes** from batch lifetime bugs
- **Converts fatal errors to handled errors** (graceful failures)
- **Prevents concurrent decode** (re-entrancy protection)
- **Provides diagnostic logs** for any remaining issues

The critical fix is that **batch creation, population, decode, and freeing all happen in one synchronous scope on llamaQueue**, ensuring the batch cannot be freed before `llama_decode` completes.

## Related

^[source-materials/mirrors/doctrine/BATCH_LIFETIME_FIX_189.md]
