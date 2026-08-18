---
title: CRASH_FIX_IMPLEMENTATION_STATUS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CRASH_FIX_IMPLEMENTATION_STATUS.md"]
updated: 2026-07-24
---

# Crash Fix Implementation Status

## ✅ Implemented

### A1: Ring Buffer for Llama Logs ✅

- `LlamaLogRingBuffer` struct with file persistence
- Flushes on crash/background/error
- Located: lines 22-77 in `LlamaEngine.swift`

### A2: Decode Frame Logging ✅

- Comprehensive logging before each decode call
- Includes: requestId, threadId, isMetalEnabled, n_ctx, nPast, nTokensToDecode, phase, etc.
- Located: lines 2616-2645 in `LlamaEngine.swift`

### A3: Capture Last 50 Llama Logs ✅

- `getAllLlamaLogs()` function implemented
- Captured on all failure paths
- Located: lines 373-375, 2468-2472, 2647-2652

### B1: Hard Context Safety Gate ✅

- Computes `required = nPast + tokens.count`
- Blocks decode if `required > n_ctx - safetyMargin`
- Fails gracefully with error (no crash)
- Located: lines 2455-2480

### B2: Per-Session nPast Tracking ✅

- `SessionState` struct tracks nPast per session
- Prevents double-increment
- Located: lines 876-920

### B3: Max Tokens Cap ✅

- `maxNewTokens` parameter exists
- EOS/stop token handling present
- Located: throughout generation code

### C2: Enhanced State Machine ✅

- `GenerationState` enum: idle, prefill, generating, finishing
- `activeRequestId` tracking
- Located: lines 865-874

### C3: Streaming Callback Safety ✅

- Callbacks run on main queue
- Immutable payloads

### D1: Cancellation Token ✅

- `isCancelled` checks in decode loop
- Located: throughout generation code

### E1: GPU Layer Validation ✅

- Validates `n_gpu_layers` within bounds
- Located: in `InferenceConfig.create()`

### F1: Token Buffer Validation ✅

- Validates batch pointers are not nil
- Validates token IDs are non-negative
- Located: lines 2608-2691

### F2: Prompt Tokenization Limits ✅

- Token budgeter exists
- Located: `TokenBudgeter` class

### G1: Per-Session KV Cache ✅

- Session state tracking
- KV cache cleared per session

### G2: Reset Logic ✅

- `resetSession()` clears KV cache and resets nPast
- Located: throughout code

### H1: Trap Fatal Errors ✅

- All decode failures return gracefully
- No crashes from error paths

### H2: Automatic Fallback Ladder ⚠️ PARTIAL

- Context overflow suggests fallbacks
- Need: automatic retry with smaller batch/ctx/CPU

---

## ❌ Missing / Needs Fix

### C1: Serial llamaQueue for ALL Operations ❌ CRITICAL

**Problem**: Code uses `llamaLock` (NSLock) instead of `llamaQueue` (DispatchQueue)
**Impact**: NSLock provides mutual exclusion but not serialization. All llama operations should be on a serial queue.

**Current State**:

- `llamaQueue` exists (line 845) but is not used
- All llama operations use `llamaLock` (NSLock)
- `safeDecode` is called with `llamaLock` held

**Required Fix**:

1. Replace all `llamaLock.lock()` / `llamaLock.unlock()` with `llamaQueue.async` / `llamaQueue.sync`
2. Ensure `safeDecode` is only called from `llamaQueue`
3. Add `assertOnLlamaQueue()` calls to all llama operations

**Files to Fix**:

- All places using `llamaLock` (29 occurrences found)
- `safeDecode` function
- Model load/unload
- Context creation/free
- Sampling
- KV cache operations
- Adapter load/unload

### H2: Automatic Fallback Ladder ⚠️ PARTIAL

**Current**: Context overflow logs suggestions
**Needed**: Automatic retry with:

1. Smaller `n_batch`
2. Smaller `n_ctx`
3. CPU-only mode

---

## 🔧 Implementation Priority

1. **C1: Serial llamaQueue** - CRITICAL (prevents concurrency crashes)
2. **H2: Automatic Fallback** - HIGH (prevents crashes from resource limits)

---

## 📝 Notes

- Most safety checks are already in place
- The main remaining issue is ensuring all llama operations are serialized via `llamaQueue` instead of `llamaLock`
- The codebase is well-instrumented with logging
- Context overflow protection is robust

## Related

^[{src_rel}]
