---
title: INFERENCE_PERFORMANCE_ANALYSIS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/INFERENCE_PERFORMANCE_ANALYSIS.md"]
updated: 2026-07-24
---

# Inference Performance Analysis & Fix Proposal

## Problem Summary

Inference is taking **15-30+ seconds per decode chunk**, making the app unusable. Despite logs showing:

- ✅ 99/99 layers on GPU (100% GPU acceleration)
- ✅ Metal GPU active
- ✅ Model loaded successfully

The actual decode performance suggests **CPU fallback** or **Metal not actually working**.

## Root Cause Analysis

### 1. **Chunk Size Too Small** (Primary Issue)

- Current: `chunkSize = actualContextBatchSize = 64` tokens
- Impact:
  - 191 tokens = 3 chunks × 15-30s = **45-90 seconds total**
  - 2864 tokens = 45 chunks × 15-30s = **11-22 minutes total** (unacceptable)
- Code location: `LlamaEngine.swift:3058`

### 2. **Metal GPU May Not Be Active Despite Logs**

- Logs show `n_gpu_layers=99` but decode speed suggests CPU
- Expected GPU speed: ~50 tokens/ms
- Actual speed: ~0.1-0.5 tokens/ms (100-500x slower than expected)
- Code location: `LlamaEngine.swift:3258-3260`

### 3. **Blocking Decode Call**

- `llama_decode()` is a blocking C function call
- No async/await or timeout mechanism
- Lock held during entire decode (`llamaLock.lock()` at line 3231)
- Code location: `LlamaEngine.swift:1655, 3233`

### 4. **Context Window Reduced**

- Logs show `n_ctx=2048` instead of configured `8192`
- Suggests memory pressure causing dynamic resizing
- May be causing Metal to fall back to CPU
- Code location: Context resizer logic

### 5. **Excessive Validation Overhead**

- Multiple guard statements and validations before each decode
- Lock acquired twice (once for batch building, once for decode)
- Code location: `LlamaEngine.swift:3098-3235`

## Evidence from Logs

```
[2026-01-23T15:40:13Z] [INFO] [LlamaEngine] [9C2F8A45] About to decode chunk 1 (batch size: 64, Metal: YES)
[2026-01-23T15:40:36Z] [INFO] LlamaEngine: getDiagnosticStatus...  // 23 seconds later
[2026-01-23T15:41:00Z] [INFO] LlamaEngine: getDiagnosticStatus...  // 24 seconds later
```

**No "Decode call completed" log** - suggests the decode is hanging or extremely slow.

## Proposed Solutions

### Solution 1: Increase Chunk Size (Quick Win - High Impact)

**Problem**: 64-token chunks are too small, causing excessive decode calls.

**Fix**: Use larger chunks when Metal GPU is active:

```swift
// At line 3058, replace:
let chunkSize = Int(actualContextBatchSize)  // Currently 64

// With:
let chunkSize: Int = {
    if isUsingMetal && actualGpuLayers >= 80 {
        // Metal GPU can handle larger batches efficiently
        return min(512, Int(actualContextBatchSize) * 4)  // 256 tokens for 64 batch
    } else {
        // CPU or partial GPU - use smaller chunks
        return Int(actualContextBatchSize)
    }
}()
```

**Expected Impact**:

- 191 tokens: 3 chunks → 1 chunk = **15-30s → 5-10s** (3x faster)
- 2864 tokens: 45 chunks → 6 chunks = **11-22min → 1.5-3min** (7x faster)

### Solution 2: Verify Metal GPU Actually Working

**Problem**: Logs say Metal is active, but performance suggests CPU.

**Fix**: Add performance-based Metal verification:

```swift
// After decode at line 3254, add:
#if !targetEnvironment(simulator) && canImport(Metal)
let expectedGpuSpeed = 50.0  // tokens/ms on GPU
let actualSpeed = tokensPerMs
let speedRatio = actualSpeed / expectedGpuSpeed

if isUsingMetal && speedRatio < 0.1 {
    // Metal flag is set but performance suggests CPU
    let warning = "[LlamaEngine] [\(requestId.uuidString.prefix(8))] METAL_PERFORMANCE_MISMATCH: Metal flag=true but speed=\(String(format: "%.2f", actualSpeed)) tokens/ms (expected ~\(expectedGpuSpeed)). GPU may not be active."
    CrashLogger.shared.log(warning, level: "ERROR")

    // Force CPU fallback for next request
    // (Could implement automatic fallback here)
}
#endif
```

### Solution 3: Remove Redundant Lock Acquisition

**Problem**: Lock acquired twice (batch building + decode).

**Fix**: Single lock scope:

```swift
// At line 3098, change from:
llamaLock.lock()
defer { llamaLock.unlock() }
// ... batch building ...

// Then at 3231:
llamaLock.lock()  // REDUNDANT - already locked
let (decodeSuccess, shouldRetryWithCPU) = safeDecode(...)
llamaLock.unlock()

// To: Keep single lock scope
llamaLock.lock()
defer { llamaLock.unlock() }

// Build batch
// ... batch building code ...

// Decode (lock already held)
let (decodeSuccess, shouldRetryWithCPU) = safeDecode(...)
```

### Solution 4: Add Decode Timeout & Progress Logging

**Problem**: No visibility into decode progress, no timeout.

**Fix**: Add timeout and progress logging:

```swift
// At line 3231, replace blocking decode with:
let decodeStartTime = Date()
let timeoutSeconds: TimeInterval = isUsingMetal ? 5.0 : 30.0  // GPU should be fast

// Use DispatchQueue for timeout
let decodeSemaphore = DispatchSemaphore(value: 0)
var decodeResult: (Bool, Bool) = (false, false)

DispatchQueue.global(qos: .userInitiated).async {
    decodeResult = safeDecode(ctx, promptBatch, requestId: requestId, operation: "prompt_chunk_\(chunkNum)")
    decodeSemaphore.signal()
}

let timeoutResult = decodeSemaphore.wait(timeout: .now() + timeoutSeconds)
let decodeEndTime = Date()
let decodeCallMs = Int(decodeEndTime.timeIntervalSince(decodeStartTime) * 1000)

if timeoutResult == .timedOut {
    let timeoutMsg = "[LlamaEngine] [\(requestId.uuidString.prefix(8))] DECODE_TIMEOUT: Chunk \(chunkNum) exceeded \(timeoutSeconds)s timeout"
    CrashLogger.shared.log(timeoutMsg, level: "ERROR")
    // Handle timeout (retry with CPU, or fail)
    DispatchQueue.main.async {
        completion(.failure(LlamaError.inferenceError("Decode timeout")))
    }
    return
}
```

### Solution 5: Optimize Context Window Management

**Problem**: Context resized to 2048, may be causing Metal issues.

**Fix**: Ensure context size is appropriate for Metal:

```swift
// When creating context, ensure n_ctx is sufficient
// Check if context resizer is reducing too aggressively
// Add logging to track context size changes
```

## Implementation Priority

1. **Solution 1** (Increase chunk size) - **IMMEDIATE** - 5 min fix, 3-7x speedup
2. **Solution 2** (Metal verification) - **HIGH** - 10 min fix, identifies root cause
3. **Solution 3** (Remove redundant lock) - **MEDIUM** - 5 min fix, reduces overhead
4. **Solution 4** (Timeout) - **MEDIUM** - 15 min fix, prevents hangs
5. **Solution 5** (Context optimization) - **LOW** - 30 min fix, may not be issue

## Testing Plan

1. **Baseline**: Measure current decode time for 191-token prompt
2. **Apply Solution 1**: Measure decode time with larger chunks
3. **Verify**: Check logs for "METAL_PERFORMANCE_MISMATCH" warnings
4. **Stress test**: Test with 2000+ token prompts
5. **Monitor**: Watch for timeout errors or Metal fallbacks

## Expected Results

After implementing Solutions 1-3:

- **191 tokens**: 45-90s → **5-15s** (3-6x faster)
- **2864 tokens**: 11-22min → **1.5-4min** (4-7x faster)
- Better visibility into Metal GPU status
- Reduced lock contention

## Risk Assessment

- **Solution 1**: Low risk - only increases batch size when Metal is active
- **Solution 2**: Low risk - only adds logging/verification
- **Solution 3**: Low risk - removes redundant code
- **Solution 4**: Medium risk - timeout handling needs careful error recovery
- **Solution 5**: Low risk - mostly logging

## Next Steps

1. Implement Solution 1 immediately (5 minutes)
2. Test on device with real prompts
3. If still slow, implement Solution 2 to diagnose Metal issue
4. Apply remaining solutions based on findings

## Related

^[{src_rel}]
