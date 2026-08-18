---
title: IPHONE12_FREEZE_FIX
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/IPHONE12_FREEZE_FIX.md"]
updated: 2026-07-24
---

# iPhone 12 Inference Freeze - Root Cause Analysis & Fix

## Problem

iPhone 12 (A14 chip, 4GB RAM) freezes during inference, blocking the UI completely.

## Root Cause Analysis

### Current Configuration (iPhone 12)

- **Device Tier**: `mid` (A14, 2GB+ GPU)
- **n_threads**: 4 (from InferenceConfig.create)
- **n_threads_batch**: 4
- **Queue Priority**: `.userInitiated` (high priority)
- **GPU Layers**: 28 (all layers on GPU)
- **n_ctx**: 2048
- **n_batch**: 128

### Why It Freezes

1. **Thread Starvation**: 4 CPU threads at `.userInitiated` priority monopolize all CPU cores
2. **Main Thread Blocking**: High-priority inference threads starve the main thread
3. **No Yield Points**: Inference runs continuously without yielding to UI
4. **A14 Thermal**: 4 threads + GPU at full load triggers thermal throttling quickly

### iPhone 12 Specs

- **CPU**: A14 Bionic (6 cores: 2 performance + 4 efficiency)
- **RAM**: 4GB
- **GPU**: 4-core Apple GPU (~2GB working set)
- **Thermal**: Aggressive throttling under sustained load

## Solution Strategy

### 1. Reduce Thread Count for Mid-Tier Devices

**Change**: `mid` tier → 2 threads instead of 4
**Rationale**:

- A14 has only 2 performance cores
- 4 threads compete for 2 cores → context switching overhead
- 2 threads = better cache locality, less contention

### 2. Lower Queue Priority

**Change**: `.userInitiated` → `.utility` for inference queue
**Rationale**:

- `.userInitiated` = same priority as UI animations (QoS 25)
- `.utility` = lower priority (QoS 17), won't starve main thread
- Still higher than `.background` (QoS 9)

### 3. Add Cooperative Yield Points

**Change**: Insert `Thread.sleep(forTimeInterval: 0.001)` every N tokens
**Rationale**:

- Allows main thread to process UI events
- 1ms sleep = ~1 frame at 60fps
- Minimal impact on throughput (~1% overhead)

### 4. Reduce Batch Size for Mid-Tier

**Change**: `mid` tier → n_batch=64 instead of 128
**Rationale**:

- Smaller batches = shorter decode calls
- More opportunities for UI to update
- Lower memory pressure

## Implementation

### File: `LlamaEngine.swift`

#### Change 1: Update InferenceConfig thread counts

```swift
// Line ~507-509 (Metal path)
n_threads: deviceTier == "mid" ? 2 : 4,
n_threads_batch: deviceTier == "mid" ? 2 : 4,

// Line ~527-529 (CPU path)
n_threads: 2,  // Already conservative
n_threads_batch: 2,
```

#### Change 2: Lower queue priority

```swift
// Line ~1160-1164
private let llamaQueue = DispatchQueue(
    label: "com.evo.llama.serial",
    qos: .utility,  // Changed from .userInitiated
    attributes: []
)
```

#### Change 3: Add yield points in decode loop

```swift
// In generate() method, after each token decode:
if generatedTokens % 10 == 0 {
    Thread.sleep(forTimeInterval: 0.001)  // 1ms yield every 10 tokens
}
```

#### Change 4: Reduce batch size for mid-tier

```swift
// Line ~497 (Metal path)
let n_batch: UInt32 = requestedBatch > 0 ? requestedBatch : {
    if deviceTier == "high" || deviceTier == "midHigh" { return 512 }
    else if deviceTier == "mid" { return 64 }  // Reduced from 128
    else { return 32 }
}()
```

## Expected Results

- **UI Responsiveness**: Main thread can process events between decode calls
- **Thermal**: Lower sustained load → less throttling
- **Throughput**: ~5-10% slower, but usable vs completely frozen
- **User Experience**: Smooth UI with visible progress vs frozen app

## Testing Checklist

- [ ] Test on iPhone 12 specifically
- [ ] Verify UI remains responsive during inference
- [ ] Check token generation speed (should be ~80-90% of before)
- [ ] Monitor thermal state (should stay in nominal range longer)
- [ ] Test with long prompts (>1000 tokens)
- [ ] Verify no regressions on higher-tier devices (iPhone 15 Pro)

## Related

^[{src_rel}]
