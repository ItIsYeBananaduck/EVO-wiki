---
title: ADR_LLAMA_INFERENCE_OPTIMIZATION
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/ADR_LLAMA_INFERENCE_OPTIMIZATION.md"]
updated: 2026-07-24
---

# Architecture Decision Record: Llama Inference Optimization

## Status

Accepted

## Context

On-device llama.cpp (Metal) inference is slow due to:

- System prompt bloat (~2811 tokens, 98% of total prompt)
- Tiny prefill chunking (32 tokens → ~90 decode calls)
- Inconsistent n_batch configuration (guards cap to 64/128, code assumes 512)
- Fixed large context (8192) even for short prompts
- Unnecessary KV cache clears (full prefill every turn)

## Decision

Implement a layered optimization strategy:

### 1. Prompt Architecture (PromptBuilder)

- **Base system prompt**: ≤200 tokens (identity, safety, minimal behavior)
- **Tool schema**: On-demand injection only when `needsTools()` returns true
- **Token budgeting**: Hard cap at 80% of context size, truncate conversation history first
- **Mode-based building**: BASE mode (no tools) vs TOOL mode (with tools)

### 2. Prefill Optimization

- **Chunk size = actual n_batch**: Use `actualContextBatchSize` (64/128), not fixed 32
- **Single source of truth**: All code paths reference `actualContextBatchSize` property
- **Eliminate 32-token chunks**: Only use as emergency fallback if decode fails

### 3. Dynamic Context Window

- **Start small**: Initial context = 2048 tokens (faster, less memory)
- **Grow on demand**: Resize to 3072/4096/6144/8192 only when token estimate exceeds current
- **Context resizer**: Manages resizing, reuses model handle (no reload from disk)
- **Device tier aware**: Respects Metal vs CPU limits

### 4. KV Cache Strategy

- **Retain across turns**: Keep KV cache for multi-turn conversations (append-only)
- **Clear only when**: New conversation starts OR context resized OR explicit reset
- **n_past continuation**: Continue from previous position if cache kept

### 5. Configuration Consistency

- **Single source of truth**: `actualContextBatchSize` property stores real n_batch used
- **Metal-safe guards**: Applied once during context creation, stored in property
- **All code uses property**: No hardcoded 512, no mismatched assumptions

### 6. Instrumentation & Guards

- **Timing logs**: Tokenization, prefill (total + per-chunk), generation tokens/sec
- **GPU verification**: Log actual GPU layers, warn if <80%
- **Requested vs actual**: Log n_ctx, n_batch, GPU layers, flash_attn discrepancies
- **Performance thresholds**: Warn if prefill >500ms per chunk, GPU layers <80%

## Consequences

### Positive

- **3-4x faster prefill**: Fewer decode calls (90 → 20-30 chunks)
- **70-85% token reduction**: System prompt from ~2800 to <250 tokens
- **Better memory usage**: Start with smaller context, grow only when needed
- **Faster multi-turn**: KV cache reuse avoids full prefill
- **Consistent behavior**: Single source of truth prevents config mismatches

### Negative

- **Context resizing overhead**: Recreating context when growing (acceptable trade-off)
- **Tool loading complexity**: On-demand loading requires `needsTools()` gate logic
- **KV cache memory**: Retaining cache uses more memory (acceptable for speed)

### Risks & Mitigations

- **Risk**: Context resize might fail on low-memory devices
  - **Mitigation**: Fallback to smaller context, log warning
- **Risk**: Tool detection might miss some cases
  - **Mitigation**: `lastRequestUsedTools` flag tracks previous usage
- **Risk**: KV cache might grow unbounded
  - **Mitigation**: Conversation history limited to last 10 turns

## Implementation Notes

- All changes are backward compatible (graceful degradation)
- Metal-safe guards remain in place (stability first)
- Instrumentation added for monitoring and debugging
- Code is structured for easy testing and regression prevention

## Related
