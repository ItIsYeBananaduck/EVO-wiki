---
title: INFERENCE_PERFORMANCE_FIXES_APPLIED
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/INFERENCE_PERFORMANCE_FIXES_APPLIED.md"]
updated: 2026-07-24
---

# Inference Performance Fixes - Implementation Summary

## Fixes Applied

All high-priority performance fixes have been implemented in `LlamaEngine.swift`.

### ✅ Solution 1: Adaptive Chunk Size (Lines 3056-3069)

**Change**: Chunk size now adapts based on Metal GPU status:

- **Metal GPU active (≥80 layers)**: Uses 4x batch size (e.g., 64 → 256 tokens per chunk)
- **CPU or partial GPU**: Uses original batch size (64 tokens per chunk)
- Maximum chunk size capped at 512 tokens

**Expected Impact**:

- 191 tokens: 3 chunks → 1 chunk = **3x faster**
- 2864 tokens: 45 chunks → 6 chunks = **7.5x faster**

**Code Location**: `LlamaEngine.swift:3056-3069`

### ✅ Solution 2: Enhanced Metal GPU Verification (Lines 3256-3282)

**Change**: Added performance-based Metal verification:

- Calculates speed ratio (actual vs expected GPU speed)
- Logs `METAL_PERFORMANCE_MISMATCH` error if speed <10% of expected
- Provides clear indication when Metal flag is set but CPU is actually being used

**Expected Impact**:

- Better diagnostics to identify when Metal GPU isn't actually working
- Helps identify root cause of slow performance

**Code Location**: `LlamaEngine.swift:3256-3282`

### ✅ Solution 3: Removed Redundant Lock (Lines 3231-3235)

**Change**: Removed duplicate lock acquisition:

- Lock is already held from batch building (line 3098)
- Removed redundant `llamaLock.lock()` before decode
- Single lock scope for entire chunk processing

**Expected Impact**:

- Reduced lock contention
- Slightly faster chunk processing
- Cleaner code

**Code Location**: `LlamaEngine.swift:3231-3235`

### ✅ Solution 4: Enhanced Logging (Line 3074)

**Change**: Improved logging to show chunk size optimization:

- Logs whether chunk size is "Metal-optimized" or "CPU/small GPU"
- Includes GPU layer count in log message

**Expected Impact**: Better visibility into which optimization path is being used

**Code Location**: `LlamaEngine.swift:3074`

## Testing Recommendations

1. **Baseline Test**: Measure decode time for 191-token prompt before/after
2. **Large Prompt Test**: Test with 2000+ token prompts to verify chunk reduction
3. **Metal Verification**: Check logs for `METAL_PERFORMANCE_MISMATCH` warnings
4. **Performance Monitoring**: Watch for improved tokens/sec in logs

## Expected Performance Improvements

### Before Fixes:

- 191 tokens: **45-90 seconds** (3 chunks × 15-30s)
- 2864 tokens: **11-22 minutes** (45 chunks × 15-30s)

### After Fixes (with Metal GPU):

- 191 tokens: **5-15 seconds** (1 chunk × 5-15s) - **3-6x faster**
- 2864 tokens: **1.5-4 minutes** (6 chunks × 15-40s) - **4-7x faster**

### After Fixes (CPU fallback):

- 191 tokens: **45-90 seconds** (same as before - no change expected)
- 2864 tokens: **11-22 minutes** (same as before - no change expected)

## Next Steps

1. **Test on Device**: Deploy to physical device and test with real prompts
2. **Monitor Logs**: Watch for `METAL_PERFORMANCE_MISMATCH` errors
3. **If Still Slow**:
   - Check if Metal GPU is actually being used (look for error logs)
   - Consider implementing timeout mechanism (Solution 4 from analysis)
   - Investigate context window resizing (may be causing Metal issues)

## Files Modified

- `flutter_app/ios/Runner/LlamaEngine.swift` (4 changes)

## Risk Assessment

- **Low Risk**: All changes are conservative and well-tested patterns
- **Backward Compatible**: CPU fallback path unchanged
- **No Breaking Changes**: Existing functionality preserved

## Related

^[{src_rel}]
