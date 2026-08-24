---
title: METAL_SAFETY_PATCH_SUMMARY
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/METAL_SAFETY_PATCH_SUMMARY.md"]
updated: 2026-07-24
---

# Metal Safety Patch for Phi-4-mini on iPhone

## Overview

This patch implements Metal-safe guards and runtime fallback policies to stabilize Phi-4-mini inference on real iPhone devices while preserving high context (n_ctx = 8192).

## Changes Made

### 1. Metal-Safe Guards (Context Creation)

**Location**: `LlamaEngine.swift` lines ~1320-1335

- **Flash Attention**: Force disabled for Metal (device builds only)
  - Phi-4-mini crashes with flash_attn enabled on Metal
  - CPU mode keeps flash_attn enabled for performance
  - Simulator uses CPU path (flash_attn disabled)

- **Batch Size Cap**: Limited to 128 for Metal (configurable to 64 if needed)
  - High-end tier normally uses n_batch = 512
  - Metal-safe cap: 128 (can be reduced to 64 by changing `metalSafeBatchCap`)
  - Preserves n_ctx = 8192 (unchanged)

- **GPU Layers**: Progressive fallback (99 → 60 → 40 → CPU)
  - Attempts 99 layers first (target for high-end)
  - Falls back to 60 if 99 fails
  - Falls back to 40 if 60 fails
  - Falls back to CPU (n_gpu_layers = 0) if all GPU attempts fail

### 2. Runtime Fallback Policy

**Location**: `LlamaEngine.swift` lines ~1431-1458 (safeDecode helper)

- **Decode Failure Detection**: Wraps all `llama_decode()` calls
- **Metal Failure Logging**: Logs when Metal decode fails with recommendation for CPU fallback
- **Graceful Handling**: Does not crash app, logs error and returns failure status
- **Initialization Fallback**: Already implemented in `loadModel()` - tries CPU first, then GPU with progressive layer reduction

### 3. Context Safety Guarantees

**Location**: `LlamaEngine.swift` lines ~2354-2365

- **Serial Queue**: `generationQueue` ensures only one generation runs at a time
- **Explicit Guard**: Added state check to prevent concurrent generation attempts
- **Lock Protection**: All llama operations protected by `llamaLock` (NSLock)

### 4. Structured Logging

**Location**: `LlamaEngine.swift` lines ~2430-2452

Logs the following in ONE structured line before each generation:

- Device model (e.g., "iPhone")
- Device tier (high/midHigh/mid/lowMid/low)
- usingMetal (true/false)
- n_ctx (context size, preserved at 8192)
- n_batch (batch size, may be capped for Metal)
- n_gpu_layers (actual GPU layers in use)
- flash_attn (ENABLED/DISABLED)
- Model filename and file size

**Example log line**:

```
[LlamaEngine] [abc12345] GENERATION_START | device=iPhone | tier=high | usingMetal=true | n_ctx=8192 | n_batch=128 | n_gpu_layers=99 | flash_attn=DISABLED | model=alice-phi4-mini-q4.gguf | size=2491874016 bytes
```

## Why Phi-3 Tolerated 99 GPU Layers But Phi-4 Does Not

### Architectural Differences

1. **Model Architecture**:
   - **Phi-3**: Uses a more conservative attention mechanism that is more tolerant of Metal's memory management quirks
   - **Phi-4-mini**: Uses optimized attention patterns that may trigger edge cases in Metal's buffer allocation

2. **Flash Attention Implementation**:
   - **Phi-3**: Flash attention was more stable on Metal, or the model's attention patterns didn't stress Metal's flash attention implementation
   - **Phi-4-mini**: Flash attention on Metal causes SIGABRT crashes, likely due to:
     - More aggressive memory reuse patterns
     - Different tensor shapes that trigger Metal buffer allocation bugs
     - Kernel compilation issues with Phi-4's attention patterns

3. **Memory Pressure**:
   - **Phi-3**: Lower memory pressure at same context size (smaller model or more efficient memory layout)
   - **Phi-4-mini**: Higher memory pressure triggers Metal's memory management to fail more aggressively

4. **Batch Processing**:
   - **Phi-3**: Tolerated larger batch sizes (512) on Metal
   - **Phi-4-mini**: Requires smaller batches (128 or 64) to avoid Metal buffer overflow

### Root Cause Hypothesis

The crashes are likely caused by:

1. **Metal Buffer Allocation**: Phi-4-mini's tensor shapes trigger bugs in Metal's buffer allocation when flash attention is enabled
2. **Kernel Compilation**: Metal shader compilation for Phi-4's attention patterns may have edge cases
3. **Memory Fragmentation**: Higher memory pressure causes Metal to fail buffer allocation more frequently

**Solution**: Disable flash attention and reduce batch size for Metal, while keeping high context (8192) which is critical for tool calling.

## Test Matrix

### Configuration Matrix

| n_batch | flash_attn | n_gpu_layers | n_ctx | Expected Result                       |
| ------- | ---------- | ------------ | ----- | ------------------------------------- |
| 128     | OFF        | 99           | 8192  | ✅ Primary target (high-end)          |
| 128     | OFF        | 60           | 8192  | ✅ Fallback if 99 fails               |
| 128     | OFF        | 40           | 8192  | ✅ Fallback if 60 fails               |
| 64      | OFF        | 99           | 8192  | ⚠️ If 128 still crashes, reduce to 64 |
| 64      | OFF        | 60           | 8192  | ⚠️ If 128 still crashes, reduce to 64 |
| 64      | OFF        | 40           | 8192  | ⚠️ If 128 still crashes, reduce to 64 |
| \*      | OFF        | 0            | 8192  | ✅ CPU fallback (always works)        |

### Testing Procedure

1. **Initial Test (n_batch=128, flash_attn=OFF, n_gpu_layers=99)**:
   - Load model on real iPhone (TestFlight)
   - Run 10 generations
   - Check logs for "METAL_FAILURE" or crashes
   - If stable: ✅ Done
   - If crashes: Proceed to step 2

2. **Fallback Test (n_batch=128, flash_attn=OFF, n_gpu_layers=60)**:
   - System automatically tries 60 layers if 99 fails
   - Check logs for successful load with 60 layers
   - Run 10 generations
   - If stable: ✅ Done
   - If crashes: Proceed to step 3

3. **Further Fallback (n_batch=128, flash_attn=OFF, n_gpu_layers=40)**:
   - System automatically tries 40 layers if 60 fails
   - Check logs for successful load with 40 layers
   - Run 10 generations
   - If stable: ✅ Done
   - If crashes: Proceed to step 4

4. **Reduce Batch Size (n_batch=64)**:
   - Edit `LlamaEngine.swift` line ~1329: Change `metalSafeBatchCap` from 128 to 64
   - Rebuild and test
   - Repeat steps 1-3 with n_batch=64

5. **CPU Fallback (n_gpu_layers=0)**:
   - System automatically falls back to CPU if all GPU attempts fail
   - Check logs for "CPU fallback" message
   - Run 10 generations
   - Should always work (slower but stable)

### Success Criteria

- ✅ No SIGABRT crashes during generation
- ✅ n_ctx = 8192 preserved (check logs)
- ✅ Model loads successfully (check logs for GPU layer count)
- ✅ Generations complete without errors
- ✅ Structured logging shows correct Metal-safe config

### Monitoring

Watch for these log patterns:

**Success**:

```
[LlamaEngine] ✓ GPU load succeeded with 99 layers
[LlamaEngine] METAL-SAFE: flash_attn=OFF, n_batch capped to 128
[LlamaEngine] GENERATION_START | usingMetal=true | n_gpu_layers=99 | flash_attn=DISABLED
```

**Fallback**:

```
[LlamaEngine] GPU load failed with 99 layers, trying next...
[LlamaEngine] ✓ GPU load succeeded with 60 layers
```

**Failure**:

```
[LlamaEngine] METAL_FAILURE: Generation decode failed, CPU fallback needed
[LlamaEngine] All GPU attempts failed, falling back to CPU
```

## Code Locations

- **Metal-safe guards**: Lines ~1320-1335
- **GPU layer fallback**: Lines ~1195-1245
- **Safe decode wrapper**: Lines ~1431-1458
- **Structured logging**: Lines ~2430-2452
- **Context safety guard**: Lines ~2354-2365

## Next Steps if Issues Persist

1. **Reduce n_batch to 64**: Change `metalSafeBatchCap` from 128 to 64
2. **Further reduce GPU layers**: Add 30, 20 to fallback sequence
3. **Investigate Metal version**: Check if Metal framework version affects stability
4. **Profile memory usage**: Use Instruments to identify memory pressure points

## Notes

- **n_ctx = 8192 is preserved** - This is non-negotiable for tool calling support
- **Performance impact**: Disabling flash_attn and reducing batch size may slow generation by ~10-20%
- **CPU fallback**: Always available as last resort (slower but stable)

## Related
