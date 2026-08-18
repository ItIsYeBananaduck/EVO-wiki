---
title: CRASH_ANALYSIS_156
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CRASH_ANALYSIS_156.md"]
updated: 2026-07-24
---

# Crash Analysis: Build 156 - SIGABRT in llama_decode

## Crash Summary

**Exception Type**: `EXC_CRASH (SIGABRT)`
**Triggered by**: Thread 5
**Location**: `LlamaEngine.safeDecode(_:_:requestId:operation:)` at line 1666
**Called from**: `LlamaEngine._processGeneration(...)` at line 3261
**Build**: 156 (after batch size optimization)

## Stack Trace Analysis

The crash shows **much deeper llama.cpp stack trace** than before:

```
3-13: llama.cpp internal calls (11 levels deep)
14: Runner LlamaEngine.safeDecode(...) + 848 (LlamaEngine.swift:1666)
15: Runner LlamaEngine._processGeneration(...) + 32684 (LlamaEngine.swift:3261)
```

This suggests the assertion failure is **deeper in llama.cpp**, possibly:

- Batch structure validation
- Position sequence validation
- Memory bounds checking
- Context state validation

## Root Cause Hypothesis

### Issue 1: Batch Size Still Too Large

Even though we're using `actualContextBatchSize`, the value might be **256** (from our optimization), which may exceed llama.cpp's internal limits or Metal GPU's actual capabilities.

**Evidence**:

- We set `n_batch = 256` for 99/99 GPU layers
- But llama.cpp might have hard limits or Metal might not support 256

### Issue 2: Batch Allocation vs Context Mismatch

The batch is allocated with `chunkSize = actualContextBatchSize`, but if the context was created with a different `n_batch` value (due to context resizing or other factors), there could be a mismatch.

### Issue 3: Position Sequence Issue

The positions are set as `nPast + Int32(i)`, but if `nPast` is very large or the sequence is invalid, llama.cpp might abort.

### Issue 4: Memory Corruption

Large batch sizes (256) might cause memory issues on device, leading to corruption that triggers assertions.

## Proposed Fixes

### Fix 1: Conservative Batch Size (IMMEDIATE)

Reduce the maximum batch size to a proven safe value:

```swift
let metalSafeBatchCap: UInt32 = {
    if actualGpuLayers >= 99 {
        // Even with full GPU, be conservative - 128 is safer than 256
        return min(128, batchSize)  // Reduced from 256
    } else if actualGpuLayers >= 80 {
        return min(128, batchSize)
    } else {
        return 64
    }
}()
```

### Fix 2: Validate Context n_batch Before Use

Add validation to ensure context's actual n_batch matches what we think it is:

```swift
// After context creation, verify n_batch
let actualCtxBatch = llama_n_batch(ctx)  // Get actual value from context
if actualCtxBatch != ctxParams.n_batch {
    let warnMsg = "[LlamaEngine] WARNING: Context n_batch mismatch - requested \(ctxParams.n_batch), got \(actualCtxBatch)"
    CrashLogger.shared.log(warnMsg, level: "WARN")
    actualContextBatchSize = actualCtxBatch  // Use actual value
}
```

### Fix 3: Add Position Validation

Validate that positions are within context bounds:

```swift
// Before decode, validate positions
for i in 0..<Int(promptBatch.n_tokens) {
    let pos = promptBatch.pos[i]
    guard pos >= 0 && pos < Int32(contextResizer?.getCurrentSize() ?? 8192) else {
        let errorMsg = "[LlamaEngine] ERROR: Invalid position \(pos) at index \(i) (context size: \(contextResizer?.getCurrentSize() ?? 0))"
        CrashLogger.shared.log(errorMsg, level: "ERROR")
        // Handle error
        return
    }
}
```

### Fix 4: Reduce Batch Size Gradually

Start with smaller batches and only increase if stable:

```swift
let metalSafeBatchCap: UInt32 = {
    if actualGpuLayers >= 99 {
        // Start conservative - 128 instead of 256
        // Can increase later if stable
        return min(128, batchSize)
    } else if actualGpuLayers >= 80 {
        return min(128, batchSize)
    } else {
        return 64
    }
}()
```

## Recommended Immediate Fix

**Reduce maximum batch size from 256 to 128**:

```swift
// Line 1438: Change from
return min(256, batchSize)  // Too aggressive

// To:
return min(128, batchSize)  // Safer, still 2x improvement over 64
```

This will:

- ✅ Still provide 2x performance improvement (128 vs 64)
- ✅ Much safer than 256
- ✅ Less likely to hit llama.cpp limits
- ✅ Better memory safety

## Testing Plan

1. Deploy with 128 max batch size
2. Monitor for crashes
3. If stable, can try 192 or 256 later
4. Check logs for actual `n_batch` values used

## Related

^[{src_rel}]
