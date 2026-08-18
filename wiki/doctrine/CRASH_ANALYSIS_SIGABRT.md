---
title: CRASH_ANALYSIS_SIGABRT
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CRASH_ANALYSIS_SIGABRT.md"]
updated: 2026-07-24
---

# Crash Analysis: SIGABRT in llama_decode

## Crash Summary

**Exception Type**: `EXC_CRASH (SIGABRT)`
**Triggered by**: Thread 7
**Location**: `LlamaEngine.safeDecode(_:_:requestId:operation:)` at line 1655
**Called from**: `LlamaEngine._processGeneration(...)` at line 3247

## Stack Trace Analysis

```
Thread 7 Crashed:
0   libsystem_kernel.dylib        SIGABRT signal
1   libsystem_pthread.dylib        pthread_kill
2   libsystem_c.dylib              abort()
3   llama                          [llama.cpp internal abort]
4   llama                          [llama.cpp internal]
5   llama                          [llama.cpp internal]
6   Runner                         LlamaEngine.safeDecode(...) + 848 (LlamaEngine.swift:1655)
7   Runner                         LlamaEngine._processGeneration(...) + 32768 (LlamaEngine.swift:3247)
```

## Root Cause

The crash occurs at **line 1655** where `llama_decode(ctx, batch)` is called. The abort originates from **within llama.cpp**, indicating:

1. **Invalid batch structure** - The batch passed to `llama_decode` has invalid parameters
2. **Batch size too large** - The increased chunk size (256 tokens) may exceed Metal GPU limits
3. **Context corruption** - The context may be in an invalid state
4. **Memory allocation failure** - Large batch may fail to allocate on device

## Likely Cause: Increased Chunk Size

**Recent Change**: We increased chunk size from 64 to 256 tokens when Metal GPU is active.

**Problem**: The batch structure may not be properly sized or validated for 256 tokens. Looking at the code:

```swift
// Line 3058-3069: New adaptive chunk size
let chunkSize: Int = {
    if isUsingMetal && actualGpuLayers >= 80 {
        return min(512, Int(actualContextBatchSize) * 4)  // 64 * 4 = 256
    } else {
        return Int(actualContextBatchSize)  // 64
    }
}()

// Line 3066: Batch allocated with chunkSize
var promptBatch = llama_batch_init(Int32(chunkSize), 0, 1)
```

**Issue**: The batch is allocated correctly, but:

1. **Metal GPU may have a hard limit** on batch size (possibly 128 or 256)
2. **Context `n_batch` parameter** may be smaller than chunk size (e.g., `n_batch=64` but trying to decode 256 tokens)
3. **Batch validation** may not catch the mismatch before calling `llama_decode`

## Evidence

1. Crash happens **during decode** (not during batch building)
2. Abort comes from **llama.cpp internals** (assertion failure)
3. Occurs with **Metal GPU active** (99/99 layers)
4. Happens at **chunk decode** (not model load)

## Fix Strategy

### Solution 1: Cap Chunk Size to Context n_batch (IMMEDIATE FIX)

The chunk size should **never exceed** the context's `n_batch` parameter:

```swift
let chunkSize: Int = {
    if isUsingMetal && actualGpuLayers >= 80 {
        // Cap at actualContextBatchSize to prevent exceeding context limits
        let metalChunkSize = min(Int(actualContextBatchSize), 256)  // Cap at 256 or batch size
        return metalChunkSize
    } else {
        return Int(actualContextBatchSize)
    }
}()
```

### Solution 2: Add Batch Size Validation Before Decode

Add validation to ensure batch size doesn't exceed context limits:

```swift
// Before calling safeDecode, add:
guard promptBatch.n_tokens <= Int(actualContextBatchSize) else {
    let errorMsg = "[LlamaEngine] [\(requestId.uuidString.prefix(8))] ERROR: Batch size \(promptBatch.n_tokens) exceeds context n_batch \(actualContextBatchSize)"
    CrashLogger.shared.log(errorMsg, level: "ERROR")
    DispatchQueue.main.async {
        completion(.failure(LlamaError.inferenceError("Batch size exceeds context limit")))
    }
    return
}
```

### Solution 3: Use Context n_batch as Maximum

Always use `actualContextBatchSize` as the maximum chunk size:

```swift
let chunkSize = Int(actualContextBatchSize)  // Use context's n_batch as limit
// Don't multiply - context is already optimized for device
```

## Recommended Fix

**Immediate**: Cap chunk size to `actualContextBatchSize` (don't multiply by 4):

```swift
// Replace lines 3058-3069 with:
let chunkSize: Int = {
    if isUsingMetal && actualGpuLayers >= 80 {
        // Use context's n_batch directly - it's already optimized for Metal
        // Don't exceed it, but can use it fully
        return Int(actualContextBatchSize)  // Typically 64-128 for Metal
    } else {
        return Int(actualContextBatchSize)
    }
}()
```

This ensures:

- ✅ Chunk size never exceeds context limits
- ✅ Metal GPU gets full batch utilization
- ✅ No assertion failures in llama.cpp
- ✅ Still faster than before (using full batch size instead of smaller chunks)

## Testing

After fix:

1. Test with Metal GPU active (99/99 layers)
2. Verify chunk size matches `actualContextBatchSize` in logs
3. Monitor for crashes during decode
4. Check performance - should still be faster than 32-token chunks

## Additional Safety

Add defensive validation before decode:

```swift
// Before safeDecode call at line 3247:
guard promptBatch.n_tokens > 0 && promptBatch.n_tokens <= Int(actualContextBatchSize) else {
    let errorMsg = "[LlamaEngine] [\(requestId.uuidString.prefix(8))] FATAL: Invalid batch size \(promptBatch.n_tokens) (max: \(actualContextBatchSize))"
    CrashLogger.shared.log(errorMsg, level: "ERROR")
    DispatchQueue.main.async {
        completion(.failure(LlamaError.inferenceError("Invalid batch size")))
    }
    return
}
```

## Related

^[{src_rel}]
