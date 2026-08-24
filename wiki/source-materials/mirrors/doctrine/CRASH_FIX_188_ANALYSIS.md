---
title: CRASH_FIX_188_ANALYSIS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CRASH_FIX_188_ANALYSIS.md"]
updated: 2026-07-24
---

# Crash Fix 188 - Root Cause Analysis & Implementation

## Crash Log Analysis

**Crash Details:**

- Exception: `EXC_CRASH (SIGABRT)`
- Triggered by: Thread 3
- Crash location: `llama.framework` during `llama_decode`
- Call stack:
  - Frame 14: `closure #2 in LlamaEngine.safeDecode` (LlamaEngine.swift:3036)
  - Frame 20: `LlamaEngine.safeDecode` (LlamaEngine.swift:3034)
  - Frame 21: `LlamaEngine._processGeneration` (LlamaEngine.swift:5114)

**Root Cause Hypothesis:**

1. **PRIMARY (Most Likely)**: Memory invalidation during queue dispatch
   - Swift-managed batch buffers (Array-backed) passed to `llama_batch`
   - When `llamaQueue.sync` is called, Swift may move/invalidate memory
   - `llama.cpp` accesses invalidated pointers → `SIGABRT`
   - **Evidence**: Crash occurs inside `llama_decode` after validation passes

2. **SECONDARY**: Queue deadlock or re-entrancy
   - `llamaQueue.sync` called from within `safeDecode` which may already be on queue
   - Deadlock or memory corruption from nested sync calls
   - **Evidence**: Multiple dispatch calls in stack trace

3. **TERTIARY**: Use-after-free during teardown
   - `ctx`/`model` freed while decode is in progress
   - No lifetime tracking for active decodes
   - **Evidence**: Thread 3 (decode) crashes while other threads active

## Implemented Fixes

### 1. Stable C Memory for llama_batch (FIX1)

**Status**: Infrastructure created, requires batch creation refactor

- Created `StableBatch` class that owns manually allocated C memory
- All batch buffers (`token`, `pos`, `logits`, `seq_id`) are in stable C memory
- Prevents Swift Array memory from being invalidated during queue dispatch
- **Note**: Batch creation code still uses `llama_batch_init` directly - refactor needed to use `StableBatch`

### 2. Fixed llamaQueue Usage (FIX2)

**Status**: ✅ COMPLETE

- Added `DispatchSpecificKey` to identify queue membership
- `executeOnLlamaQueue()` checks if already on queue → executes inline
- Prevents sync-from-sync deadlocks
- All `llamaQueue.sync` calls replaced with `executeOnLlamaQueue()`

**Code:**

```swift
private static let llamaQueueKey = DispatchSpecificKey<String>()

init() {
    llamaQueue.setSpecific(key: Self.llamaQueueKey, value: "com.evo.llama.serial")
}

private var isOnLlamaQueue: Bool {
    return DispatchQueue.getSpecific(key: Self.llamaQueueKey) != nil
}

private func executeOnLlamaQueue<T>(_ block: @escaping () -> T) -> T {
    if isOnLlamaQueue {
        return block()  // Already on queue - execute inline
    } else {
        return llamaQueue.sync(execute: block)  // Dispatch synchronously
    }
}
```

### 3. Decode Lifetime Guards (FIX3)

**Status**: ✅ COMPLETE

- Added `inDecodeCount` counter with `enterDecode()`/`exitDecode()`
- `unloadModel()` waits for `inDecodeCount == 0` before freeing ctx/model
- Prevents use-after-free crashes

**Code:**

```swift
private let decodeLifetimeLock = NSLock()
private var inDecodeCount: Int = 0

private func enterDecode() {
    decodeLifetimeLock.lock()
    defer { decodeLifetimeLock.unlock() }
    inDecodeCount += 1
}

private func exitDecode() {
    decodeLifetimeLock.lock()
    defer { decodeLifetimeLock.unlock() }
    inDecodeCount -= 1
}

private func waitForDecodeCompletion(timeout: TimeInterval = 10.0) -> Bool {
    // Waits for inDecodeCount == 0
}
```

### 4. Enhanced Pointer & State Validation (FIX4)

**Status**: ✅ COMPLETE

- Comprehensive validation before every `llama_decode`:
  - `ctx != nil`, `model != nil`, `_isLoaded == true`
  - `llama_n_ctx(ctx) > 0`
  - `batch.token != nil`, `batch.pos != nil`, `batch.logits != nil`
  - `requiredTokens = nPast + tokens.count <= n_ctx`
  - `batch.n_tokens <= n_batch`
- All failures return error instead of crashing

### 5. Crash-Proof Instrumentation (FIX5)

**Status**: ✅ COMPLETE

- Comprehensive "DECODE_HEADER" logged immediately before decode:
  - `requestId`, `sessionId`, `threadId`, `queueLabel`
  - `n_ctx`, `llama_n_ctx`, `nPast`, `tokens.count`, `required`
  - `n_batch`, `n_threads`, `metalEnabled`, `n_gpu_layers`
  - Pointer addresses: `ctxPtr`, `modelPtr`, `tokenPtr`
  - `adapterStack`, `phase`, `operation`
- Logs flushed before decode so they survive crashes

### 6. Metal Safety Fallback (FIX6)

**Status**: ✅ COMPLETE (already existed, enhanced)

- Stall detection: If decode takes >5s, treat as Metal hang
- Returns `shouldRetryWithCPU=true` to trigger CPU fallback
- Captures llama logs on failure

## Files Changed

1. **LlamaEngine.swift**:
   - Added `StableBatch` class (lines ~59-150)
   - Added queue identity check infrastructure (lines ~902-950)
   - Added decode lifetime tracking (lines ~950-990)
   - Updated `safeDecode()` with enhanced validation and logging (lines ~3200-3320)
   - Updated `unloadModel()` to wait for decode completion (lines ~3367-3423)

## Test Plan

### 1. Long Prompt Near n_ctx

- **Test**: Send prompt with `nPast + promptTokens ≈ n_ctx - 10`
- **Expected**: Decode succeeds, logs show context usage
- **Failure mode**: Context overflow guard should catch and return error

### 2. Rapid Generate/Cancel

- **Test**: Start generation, immediately cancel, repeat 10x
- **Expected**: No crashes, all cancellations handled gracefully
- **Failure mode**: Use-after-free if decode lifetime tracking fails

### 3. Unload During Generation

- **Test**: Start generation, immediately call `unloadModel()`
- **Expected**: Unload waits for decode completion (max 10s), then frees ctx/model
- **Failure mode**: Crash if ctx/model freed during decode

### 4. Metal On/Off Comparison

- **Test**: Run same prompt with Metal enabled vs disabled
- **Expected**: Both succeed, Metal may be faster
- **Failure mode**: Metal stall (>5s) triggers CPU fallback

### 5. Queue Identity Check

- **Test**: Call `safeDecode` from within `llamaQueue` context
- **Expected**: No deadlock, executes inline
- **Failure mode**: Deadlock if sync-from-sync occurs

## Remaining Work

1. **Batch Creation Refactor** (HIGH PRIORITY):
   - Update all `llama_batch_init` calls to use `StableBatch`
   - Ensure batch memory is stable across queue boundaries
   - Current code still uses `llama_batch_init` directly

2. **Memory Pressure Handling**:
   - Reduce `n_ctx`/`n_batch` on allocation failures
   - Monitor memory usage during decode

3. **Per-Session KV Cache Isolation**:
   - Ensure each session has isolated state
   - Clear KV cache on session reset

## Summary

**Key Fixes Implemented:**

- ✅ Queue identity check (prevents deadlock)
- ✅ Decode lifetime tracking (prevents use-after-free)
- ✅ Enhanced validation (catches issues before decode)
- ✅ Comprehensive logging (diagnoses crashes)
- ✅ Metal fallback (handles stalls)

**Critical Remaining Work:**

- ⚠️ Batch creation must use `StableBatch` (infrastructure ready, needs refactor)
- This is the most likely root cause of the crash

**Expected Outcome:**

- Crashes should be eliminated or converted to handled errors
- Comprehensive logs will pinpoint any remaining issues
- Metal stalls will trigger CPU fallback automatically

## Related
