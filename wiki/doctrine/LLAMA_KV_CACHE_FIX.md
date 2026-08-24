---
title: LLAMA_KV_CACHE_FIX
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/LLAMA_KV_CACHE_FIX.md
updated: 2026-07-24
---

# LlamaEngine KV Cache Position Desync Fix

## Problem Summary

The app was crashing with `SIGABRT` due to KV cache position desync:

- Error: "sequence positions remain consecutive: Y = X + 1"
- Prompt had 130 tokens, but `llama_decode` was trying to start at position 185
- KV cache last position was 129 (expected: 129, but decode tried to start at 185)

## Root Causes Identified

1. **No single-flight protection**: Multiple `generate()` calls could run concurrently on the same `llama_context`, causing KV cache state corruption
2. **Stale position tracking**: `nCur` variable could carry over values from previous generations
3. **Position increment timing**: `nCur` was incremented before `llama_decode`, but should be incremented after
4. **Missing guardrail logging**: No validation of positions before decode

## Fix Implementation

### 1. Single-Flight Locking (`LlamaEngine.swift`)

Added `NSLock` and `isGenerating` flag to ensure only one `generate()` runs at a time:

```swift
private let generationLock = NSLock()
private var isGenerating = false
```

- Concurrent calls are rejected immediately with an error
- Lock is acquired at the start of `generate()` and released in `defer` block in the async closure
- Ensures KV cache state is not corrupted by concurrent access

### 2. KV Cache Clearing

KV cache is cleared before each fresh prompt (each `generate()` starts fresh, no chat continuation):

```swift
let memory = llama_get_memory(ctx)
llama_memory_clear(memory, true)
```

- Clears all KV cache state before processing new prompt
- Ensures positions always start at 0 for fresh prompts

### 3. Position Tracking Fixes

**Before decode:**

- Positions start at 0: `batch.pos[idx] = Int32(i)` where `i` ranges from 0 to `promptTokenCount - 1`
- Track `nPast = promptTokenCount` after prompt decode completes

**During generation loop:**

- `nCur` starts at `nPast` (not `batch.n_tokens`)
- Position for new token: `batch.pos[idx] = Int32(nCur)`
- **Key fix**: `nCur` is incremented AFTER `llama_decode` succeeds, not before
- This ensures positions are always consecutive and match KV cache state

### 4. Guardrail Logging & Assertions

Added logging before each `llama_decode`:

```swift
print("[LlamaEngine] Pre-decode: promptTokenCount=\(promptTokenCount), batch.n_tokens=\(batch.n_tokens), firstPos=\(firstPos), lastPos=\(lastPos)")
assert(firstPos == 0, "First position must be 0 after KV cache clear")
assert(lastPos == promptTokenCount - 1, "Last position must be consecutive")
```

For generation loop:

```swift
print("[LlamaEngine] Pre-decode token \(tokenCount): pos=\(nCur), token=\(newToken)")
```

### 5. Simulator Performance Cap

Added conditional compilation to cap max tokens on simulator:

```swift
#if targetEnvironment(simulator)
let maxTokens = 50  // Reduced from 256 for simulator performance
#else
let maxTokens = 256
#endif
```

- Simulator is extremely slow (~107s per token)
- This prevents UI blocking during development
- Does not affect production device behavior

## Code Changes Summary

### Key Changes in `generate()` function:

1. **Single-flight guard** at function entry (lines 193-204)
2. **KV cache clear** before prompt processing (line 242)
3. **Position logging** before prompt decode (lines 273-279)
4. **nPast tracking** after prompt decode (line 282)
5. **nCur initialization** from nPast, not batch.n_tokens (line 294)
6. **Position logging** in generation loop (line 340)
7. **nCur increment** moved to AFTER decode (line 350)

### Critical Fix: Position Increment Timing

**Before (WRONG):**

```swift
batch.pos[idx] = Int32(nCur)
nCur += 1  // ❌ Incremented before decode
if llama_decode(ctx, batch) != 0 { ... }
```

**After (CORRECT):**

```swift
batch.pos[idx] = Int32(nCur)  // Position is nCur
if llama_decode(ctx, batch) != 0 { ... }
nCur += 1  // ✅ Incremented after decode succeeds
```

## Verification Checklist

### How to Reproduce the Bug (Before Fix):

1. Call `generate()` twice quickly (e.g., rapid UI taps)
2. Navigate between screens that trigger `generate()`
3. Observe crash with "sequence positions remain consecutive" error
4. Check logs for position mismatch (e.g., starting at 185 when prompt is 130 tokens)

### How to Verify the Fix:

1. ✅ **Concurrent calls rejected**: Call `generate()` twice quickly - second call should return error "Generation already in progress" instead of crashing
2. ✅ **No position mismatch**: Check logs - `firstPos=0`, `lastPos=promptTokenCount-1` for prompt decode
3. ✅ **Consecutive positions**: Generation loop logs show `pos` values incrementing consecutively (130, 131, 132, ...)
4. ✅ **No KV cache errors**: No "sequence positions remain consecutive" errors in logs
5. ✅ **No crashes**: App runs stable even with rapid generate() calls
6. ✅ **Simulator performance**: On simulator, generation stops after 50 tokens (faster, but still functional)

### Expected Log Output (After Fix):

```
[LlamaEngine] generate() called - domain: workout, message: hi...
[LlamaEngine] KV cache cleared - starting fresh generation
[LlamaEngine] Pre-decode: promptTokenCount=130, batch.n_tokens=130, firstPos=0, lastPos=129
[LlamaEngine] Prompt decoded successfully, n_past=130, starting generation...
[LlamaEngine] Pre-decode token 1: pos=130, token=1234
[LlamaEngine] Pre-decode token 2: pos=131, token=5678
...
```

## Why This Fix Works

1. **Single-flight lock** prevents concurrent access to the shared `llama_context`, eliminating race conditions
2. **KV cache clear** ensures each generation starts with a clean slate (positions 0-based)
3. **Correct position tracking** (`nCur` starts at `nPast`, increments after decode) ensures positions always match KV cache state
4. **Guardrail logging** helps catch any future regressions early
5. **Simulator cap** prevents UI blocking during development without affecting production

## Testing Recommendations

1. Test rapid successive `generate()` calls
2. Test navigation between screens that trigger generation
3. Test on both simulator (for development) and real device (for production)
4. Monitor logs for position consistency
5. Verify no crashes occur under stress

## Related

^[source-materials/mirrors/doctrine/LLAMA_KV_CACHE_FIX.md]
