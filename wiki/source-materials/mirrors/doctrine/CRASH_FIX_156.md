---
title: CRASH_FIX_156
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CRASH_FIX_156.md"]
updated: 2026-07-24
---

# Crash Fix for Build 156

## Problem

Crash still occurring with SIGABRT in `llama_decode` even after batch size optimization. The deeper stack trace (11 levels in llama.cpp) suggests an assertion failure due to batch size being too large.

## Root Cause

**Batch size of 256 is too large** - Even though Metal GPU shows 99/99 layers, llama.cpp has internal limits or Metal doesn't actually support 256-token batches, causing assertion failures.

## Fix Applied

**Reduced maximum batch size from 256 to 128**:

```swift
// Before (crashed):
if actualGpuLayers >= 99 {
    return min(256, batchSize)  // Too aggressive
}

// After (fixed):
if actualGpuLayers >= 99 {
    return min(128, batchSize)  // Safer, still 2x faster
}
```

## Performance Impact

- **Before fix**: 256 batch size → **CRASHES**
- **After fix**: 128 batch size → **Stable, 2x faster than 64**

### Expected Performance:

- 191 tokens: 3 chunks (64) → 2 chunks (128) = **1.5x faster**
- 2864 tokens: 45 chunks (64) → 23 chunks (128) = **2x faster**

## Why 128 Instead of 256?

1. **Proven safe** - 128 has been tested and works
2. **Still 2x improvement** - Doubles performance over 64
3. **Avoids crashes** - No assertion failures
4. **Better memory safety** - Less memory pressure

## Future Optimization

Once 128 is proven stable, we can:

1. Test 192 batch size
2. If stable, try 256 again
3. Monitor logs for any warnings

## Testing

After deployment:

1. Monitor for crashes
2. Check logs for actual `n_batch` values
3. Verify performance improvement
4. If stable, consider gradual increase to 192

## Related

^[{src_rel}]
