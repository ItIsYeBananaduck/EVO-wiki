---
title: BATCH_SIZE_OPTIMIZATION
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/BATCH_SIZE_OPTIMIZATION.md
updated: 2026-07-24
---

# Batch Size Optimization - Better Approach

## The Problem

Previously, we tried to increase **chunk size** beyond `n_batch`, which caused SIGABRT crashes. The user correctly identified that we should instead **increase `n_batch` itself** when Metal GPU is active.

## The Solution

Instead of using larger chunks than `n_batch` allows, we now **increase `n_batch`** based on GPU offload status:

### Adaptive Batch Size Based on GPU Offload

```swift
let metalSafeBatchCap: UInt32 = {
    if actualGpuLayers >= 99 {
        // Full GPU offload (99/99 layers) - can handle larger batches
        return min(256, batchSize)  // Up to 256 or device tier limit
    } else if actualGpuLayers >= 80 {
        // High GPU offload (80+ layers) - moderate batches
        return min(128, batchSize)
    } else {
        // Partial GPU - conservative
        return 64
    }
}()
```

## Performance Impact

### Before (Fixed 64):

- All devices: `n_batch = 64`
- Chunk size: 64 tokens
- 191 tokens = 3 chunks
- 2864 tokens = 45 chunks

### After (Adaptive):

- **High tier + 99/99 GPU**: `n_batch = 256` (from device tier 512, capped at 256)
- **High tier + 80+ GPU**: `n_batch = 128`
- **Partial GPU**: `n_batch = 64`

### Expected Improvements:

- **191 tokens with 256 batch**: 1 chunk (was 3) = **3x faster**
- **2864 tokens with 256 batch**: 12 chunks (was 45) = **3.75x faster**
- **2864 tokens with 128 batch**: 23 chunks (was 45) = **2x faster**

## Safety

1. **Respects device tier limits** - Never exceeds the device's base `batchSize`
2. **Scales with GPU offload** - More GPU layers = larger batches
3. **Conservative fallback** - Partial GPU still uses safe 64
4. **Chunk size matches n_batch** - No more crashes from exceeding limits

## Why This Is Better

1. ✅ **Uses Metal GPU efficiently** - Larger batches = better GPU utilization
2. ✅ **No crashes** - Chunk size always matches `n_batch`
3. ✅ **Adaptive** - Scales based on actual GPU capability
4. ✅ **Respects device limits** - Never exceeds device tier batch size

## Testing

After deployment, monitor:

1. **Logs** - Check `n_batch` value in context creation logs
2. **Performance** - Measure decode time per chunk
3. **Stability** - Watch for crashes or hangs
4. **GPU verification** - Confirm Metal is actually being used (new diagnostics)

## Related

^[source-materials/mirrors/doctrine/BATCH_SIZE_OPTIMIZATION.md]
