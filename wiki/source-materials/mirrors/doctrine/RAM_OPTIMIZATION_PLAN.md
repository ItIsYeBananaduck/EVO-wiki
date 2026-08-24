---
title: RAM_OPTIMIZATION_PLAN
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/RAM_OPTIMIZATION_PLAN.md"]
updated: 2026-07-24
---

# EVOtraining RAM Optimization Plan

## iPhone 12 & Constrained Device Memory Pressure Reduction

---

## 1. Executive Summary

- **Peak memory during chat inference is the primary jetsam risk** on iPhone 12 (4GB RAM, ~2.8GB app limit before jetsam)
- **Three concurrent memory consumers** create pressure spikes: LLM context/KV cache, RAG index, and Flutter UI layer
- **Model weights (~800MB-1.5GB mmap'd)** are not the issue—iOS handles mmap'd files gracefully under pressure
- **KV cache + prompt buffers + RAG embeddings** are the controllable variables; current estimates suggest 400-800MB combined at peak
- **Device tiering already exists** but isn't fully leveraged for memory budgets—iPhone 12 should get aggressive limits
- **Quick wins exist**: reduce n_ctx, trim chat history, lazy-load RAG index, evict KV cache on background
- **No changes to chat correctness or safety**—all optimizations are about when/how much data is held in RAM, not what data is processed
- **Incremental rollout via feature flags** allows safe A/B testing and instant rollback

---

## 2. Current-State Memory Map

### Estimated RAM Consumers by Stage

| Stage                  | Primary Consumers                                                            | Estimated Peak (Tier L) | Estimated Peak (Tier H) |
| ---------------------- | ---------------------------------------------------------------------------- | ----------------------- | ----------------------- |
| **Idle**               | Flutter engine, UI widgets, app state                                        | ~150MB                  | ~200MB                  |
| **Mid-workout**        | + Workout state, timers, exercise images                                     | ~200MB                  | ~250MB                  |
| **User opens chat**    | + Chat history loaded, message list rendered                                 | ~250MB                  | ~300MB                  |
| **User asks question** | + Tokenization buffers, prompt assembly                                      | ~280MB                  | ~350MB                  |
| **RAG retrieval**      | + Embedding model (TFLite ~50MB), index access (~100-200MB), query embedding | ~400MB                  | ~500MB                  |
| **Prompt assembly**    | + Full prompt string (2K-8K tokens × 4 bytes), RAG snippets                  | ~450MB                  | ~600MB                  |
| **Inference**          | + **KV cache** (n_ctx × n_layers × d_model × 2 × 2 bytes), batch buffers     | **~800-1200MB**         | **~1500-2000MB**        |
| **Rendering response** | + Streaming text buffers, markdown parsing                                   | ~850MB                  | ~1600MB                 |
| **Saving to storage**  | + Serialization buffers (transient)                                          | ~860MB                  | ~1620MB                 |
| **Backgrounding**      | Should drop to ~200MB if properly evicted                                    | ~200MB target           | ~300MB target           |

### Peak Memory Breakdown During Inference (Tier L / iPhone 12)

```
Model weights (mmap'd, not counted against dirty memory): ~800MB-1.5GB
KV cache (n_ctx=512, 28 layers, 896 dim, Q8): ~25MB per 512 tokens
  - At n_ctx=2048: ~100MB
  - At n_ctx=512: ~25MB
Prompt buffer (tokenized): ~10-50KB
Batch buffer (n_batch=32): ~5MB
RAG index (if fully loaded): ~100-200MB
Embedding model (TFLite): ~50MB
Flutter UI + Dart heap: ~150-200MB
Chat history (in-memory): ~5-20MB
Miscellaneous Swift/ObjC: ~50MB
────────────────────────────────────────
TOTAL DIRTY MEMORY AT PEAK: ~500-700MB (target)
CURRENT ESTIMATED PEAK: ~800-1200MB (too high)
```

### Critical Insight

**The KV cache scales linearly with n_ctx.** Reducing n_ctx from 2048 to 512 on iPhone 12 saves ~75MB of KV cache alone. Combined with lazy RAG loading and aggressive eviction, we can stay under 600MB peak.

---

## 3. Prioritized Action Plan

### A) No-Risk, Low-Effort (Safe Toggles & Trims)

#### A1. Reduce n_ctx on Tier L devices

- **What**: Set n_ctx=512 for iPhone 12 (currently 512, verify enforced everywhere)
- **Why**: KV cache is proportional to n_ctx; 512 vs 2048 = 4× reduction
- **Risk**: Low (already implemented, just verify)
- **Rollout**: Verify via CHUNK_CONFIG logs
- **Metrics**: Peak resident memory during inference
- **Revert**: Change config value

#### A2. Reduce n_batch on Tier L devices

- **What**: Keep n_batch=32 for iPhone 12 (already set)
- **Why**: Batch buffers scale with n_batch
- **Risk**: Low
- **Rollout**: Already in place
- **Metrics**: Inference latency, memory
- **Revert**: Config change

#### A3. Trim chat history to last N messages before prompt assembly

- **What**: Only include last 5 messages (user+assistant pairs) in prompt for Tier L
- **Why**: Reduces prompt token count, reduces KV cache usage
- **Risk**: Low (summarization can preserve context)
- **Rollout**: Feature flag `chat_history_limit_tier_l`
- **Metrics**: Prompt token count, user satisfaction (qualitative)
- **Revert**: Disable flag

#### A4. Evict KV cache on app background

- **What**: Call `llama_kv_cache_clear()` when app enters background
- **Why**: Frees KV cache memory immediately; iOS may jetsam backgrounded apps
- **Risk**: Low (stateless mode already clears KV per request)
- **Rollout**: Add to `applicationDidEnterBackground`
- **Metrics**: Background memory footprint
- **Revert**: Remove call

#### A5. Release TFLite embedding model after RAG retrieval

- **What**: Unload TFLite interpreter after embedding query, reload on next query
- **Why**: Saves ~50MB when not actively embedding
- **Risk**: Low (adds ~100ms latency on next query)
- **Rollout**: Feature flag `lazy_embedding_model`
- **Metrics**: Memory after RAG, embedding latency
- **Revert**: Disable flag

---

### B) Medium Effort (Architectural but Safe Refactors)

#### B1. Lazy-load RAG index on first query

- **What**: Don't load FAISS/HNSW index at app launch; load on first chat query
- **Why**: Saves ~100-200MB until user actually uses chat
- **Risk**: Medium (first query latency increases by ~500ms)
- **Rollout**: Feature flag `lazy_rag_index`
- **Metrics**: App launch memory, first-query latency
- **Revert**: Disable flag

#### B2. Implement RAG index memory-mapping

- **What**: Use mmap for FAISS index instead of loading into heap
- **Why**: iOS can page out mmap'd data under pressure; heap data cannot be paged
- **Risk**: Medium (requires index format change)
- **Rollout**: Staged by device tier (Tier L first)
- **Metrics**: Dirty memory vs file-backed memory
- **Revert**: Revert to heap-loaded index

#### B3. Streaming prompt assembly (avoid full string copy)

- **What**: Tokenize prompt components incrementally instead of building one giant string
- **Why**: Avoids 2× memory for string + tokenized form simultaneously
- **Risk**: Medium (requires refactor of prompt builder)
- **Rollout**: Feature flag `streaming_tokenization`
- **Metrics**: Peak memory during prompt assembly
- **Revert**: Disable flag

#### B4. Implement "memory capsule" summarization for old messages

- **What**: Summarize messages older than N turns into a single "capsule" token sequence
- **Why**: Reduces prompt size while preserving semantic context
- **Risk**: Medium (requires LLM call for summarization, or rule-based compression)
- **Rollout**: Feature flag `memory_capsules`, Tier L only initially
- **Metrics**: Prompt token count, response quality (A/B test)
- **Revert**: Disable flag

#### B5. Reduce RAG top-k from current value to 3 on Tier L

- **What**: Retrieve fewer snippets for constrained devices
- **Why**: Each snippet adds tokens to prompt; fewer snippets = smaller prompt
- **Risk**: Medium (may reduce answer quality for complex queries)
- **Rollout**: Feature flag `rag_topk_tier_l=3`
- **Metrics**: Prompt size, answer relevance (qualitative)
- **Revert**: Increase top-k

---

### C) High Impact / Optional (Bigger Changes, Still Safe)

#### C1. Implement LRU eviction for RAG snippet cache

- **What**: Cache retrieved snippets with LRU eviction; limit cache to 1MB on Tier L
- **Why**: Avoids re-embedding and re-retrieving for repeated queries
- **Risk**: Low-Medium
- **Rollout**: Feature flag `rag_snippet_cache`
- **Metrics**: Cache hit rate, retrieval latency, memory
- **Revert**: Disable flag

#### C2. Offload chat history to SQLite with lazy loading

- **What**: Don't keep full chat history in Dart heap; load messages on-demand for display
- **Why**: Chat history can grow large; SQLite is file-backed
- **Risk**: Medium (requires UI changes for virtualized list)
- **Rollout**: Feature flag `lazy_chat_history`
- **Metrics**: Dart heap size, scroll performance
- **Revert**: Disable flag

#### C3. Implement inference request coalescing

- **What**: If user sends multiple messages quickly, coalesce into single inference
- **Why**: Avoids multiple concurrent KV cache allocations
- **Risk**: Low (improves UX anyway)
- **Rollout**: Feature flag `inference_coalescing`
- **Metrics**: Concurrent inference attempts, memory spikes
- **Revert**: Disable flag

#### C4. Add memory pressure observer with graceful degradation

- **What**: Listen to `UIApplication.didReceiveMemoryWarningNotification`; trigger cleanup
- **Why**: Proactive eviction before jetsam
- **Risk**: Low
- **Rollout**: Always-on
- **Metrics**: Memory warning frequency, jetsam rate
- **Revert**: Remove observer

#### C5. Implement model weight sharing across inference sessions

- **What**: Ensure single mmap'd model instance; no duplicate loads
- **Why**: Prevents accidental double-loading of 1GB+ model
- **Risk**: Low (should already be the case; verify)
- **Rollout**: Audit + assertion
- **Metrics**: Model load count, memory
- **Revert**: N/A

#### C6. Single tokenizer instance (singleton pattern)

- **What**: Ensure only one tokenizer instance exists across the app lifecycle; reuse for all tokenization
- **Why**: Tokenizer initialization allocates vocabulary tables (~5-10MB); multiple instances waste memory
- **Risk**: Low (pure refactor, no behavior change)
- **Rollout**: Audit current usage, refactor to singleton
- **Metrics**: Tokenizer instance count, memory during prompt assembly
- **Revert**: N/A (no behavior change)

#### C7. Reusable token buffers (pool pattern)

- **What**: Pre-allocate and reuse token buffers instead of allocating new arrays per request
- **Why**: Avoids repeated malloc/free cycles; reduces heap fragmentation; eliminates transient allocations
- **Risk**: Low (requires careful lifecycle management)
- **Rollout**: Feature flag `reusable_token_buffers`
- **Metrics**: Allocation count during inference, peak memory, heap fragmentation
- **Revert**: Disable flag

#### C8. Zero-copy string handling (avoid duplication)

- **What**: Pass string slices/views instead of copying strings during prompt assembly
- **Why**: Prompt assembly currently builds multiple intermediate strings; each copy doubles memory temporarily
- **Risk**: Medium (requires Swift/Dart interop changes)
- **Rollout**: Feature flag `zero_copy_prompt`
- **Metrics**: Peak memory during prompt assembly, string allocation count
- **Revert**: Disable flag

---

## 4. Device-Tier Strategy

### Tier L (iPhone 12 class: 4GB RAM)

**Memory Budget**: 600MB peak dirty memory

| Parameter              | Value               | Rationale            |
| ---------------------- | ------------------- | -------------------- |
| n_ctx                  | 512                 | Minimize KV cache    |
| n_batch                | 32                  | Reduce batch buffers |
| Chat history in prompt | 5 messages          | Limit prompt size    |
| RAG top-k              | 3                   | Fewer snippets       |
| RAG index              | Lazy-loaded, mmap'd | Reduce heap          |
| Embedding model        | Unload after use    | Free 50MB            |
| KV cache on background | Evict immediately   | Prevent jetsam       |
| Memory capsules        | Enabled             | Compress old context |

### Tier H (iPhone 17 Pro Max class: 16GB RAM)

**Memory Budget**: 2GB peak dirty memory (comfortable headroom)

| Parameter              | Value        | Rationale                |
| ---------------------- | ------------ | ------------------------ |
| n_ctx                  | 8192         | Full context for quality |
| n_batch                | 512          | Fast inference           |
| Chat history in prompt | 20 messages  | Rich context             |
| RAG top-k              | 10           | More snippets            |
| RAG index              | Eager-loaded | Fast first query         |
| Embedding model        | Keep loaded  | No latency penalty       |
| KV cache on background | Keep for 60s | Fast resume              |
| Memory capsules        | Disabled     | Not needed               |

### Tier Detection

```swift
let memoryGB = Double(ProcessInfo.processInfo.physicalMemory) / (1024.0 * 1024.0 * 1024.0)
let tier: DeviceTier = memoryGB >= 7.5 ? .high : (memoryGB >= 5.5 ? .midHigh : (memoryGB >= 3.5 ? .mid : .low))
```

---

## 5. RAG-Specific Optimizations

### Retrieval Memory

- **Lazy-load index**: Don't load until first query
- **Mmap index file**: Use `MAP_SHARED` so iOS can evict pages
- **Unload on background**: Release index handle when app backgrounds

### Index Access Patterns

- **Single query path**: Ensure no duplicate index loads
- **Batch embedding**: If multiple queries pending, batch them

### Snippet Injection Strategy

- **Token budget**: Allocate fixed token budget for RAG (e.g., 300 tokens on Tier L)
- **Truncate snippets**: If snippets exceed budget, truncate longest first
- **Deduplicate**: Don't inject same snippet twice

### Chunking

- **Chunk size**: 256 tokens per chunk (current)
- **Overlap**: 32 tokens (current)
- **No change needed** unless memory profiling shows chunk storage is significant

### Top-k

- **Tier L**: k=3
- **Tier H**: k=10
- **Dynamic k**: Reduce k if memory pressure detected

### Caching

- **LRU cache**: 1MB on Tier L, 10MB on Tier H
- **Cache key**: Query embedding hash
- **TTL**: 5 minutes

### Eviction

- **On memory warning**: Evict all cached snippets
- **On background**: Evict cache
- **On context switch**: Keep cache (same session)

---

## 6. Prompt/Context Optimizations

### History Trimming

- **Tier L**: Last 5 user+assistant pairs
- **Tier H**: Last 20 pairs
- **Trim strategy**: Keep system prompt + trimmed history + current query

### Memory Capsules

- **Trigger**: When history exceeds N messages
- **Format**: "Previous conversation summary: [compressed summary]"
- **Generation**: Rule-based extraction of key facts (no LLM call needed)
- **Example**: "User discussed leg day workout, mentioned knee pain, set protein target to 150g"

### Summarization Strategy

- **Phase 1**: Rule-based extraction (low risk)
- **Phase 2**: LLM-generated summaries (future, higher risk)

### Token Budgets

| Component              | Tier L Budget   | Tier H Budget   |
| ---------------------- | --------------- | --------------- |
| System prompt          | 200 tokens      | 500 tokens      |
| Memory capsule         | 100 tokens      | N/A             |
| Chat history           | 150 tokens      | 1000 tokens     |
| RAG snippets           | 300 tokens      | 1000 tokens     |
| Current query          | 100 tokens      | 500 tokens      |
| **Total prompt**       | **850 tokens**  | **3000 tokens** |
| Generation headroom    | 150 tokens      | 1000 tokens     |
| **Total n_ctx needed** | **1000 tokens** | **4000 tokens** |

### Compression

- **Remove redundant whitespace** in prompt assembly
- **Use shorter system prompt** on Tier L (essential instructions only)
- **Abbreviate workout state** (e.g., "3x10 bench" instead of full JSON)

---

## 7. UI + Rendering + Storage Optimizations

### Chat List Virtualization

- **Flutter ListView.builder**: Already virtualized (verify)
- **Message widget recycling**: Ensure widgets are recycled, not recreated
- **Image thumbnails**: Load thumbnails, not full images, in chat list

### Image/Audio Attachments

- **Lazy loading**: Load attachment only when scrolled into view
- **Memory limit**: Max 3 full-size images in memory at once
- **Eviction**: Evict oldest image when limit exceeded
- **Placeholder**: Show placeholder while loading

### Text Layout

- **Avoid rich text parsing on every frame**: Cache parsed markdown
- **Limit message length display**: Truncate very long messages with "Show more"

### Persistence Batching

- **Batch writes**: Don't write to SQLite on every token; batch every 500ms
- **Async writes**: Use background isolate for persistence
- **Compression**: Compress old messages in storage (not in-memory)

### Flutter-Specific

- **Avoid `setState` on large widgets**: Use `ValueNotifier` or `Riverpod` selectors
- **Release image cache on memory warning**: `PaintingBinding.instance.imageCache.clear()`
- **Limit Dart heap**: Monitor with `dart:developer` `Service.getMemoryUsage()`

---

## 8. Test Plan

### Reproducible Scenarios

| Scenario                      | Steps                                     | Device    | Acceptance Criteria                    |
| ----------------------------- | ----------------------------------------- | --------- | -------------------------------------- |
| S1: Cold start chat           | Launch app → Open chat → Send "Hello"     | iPhone 12 | No jetsam, response in <5s             |
| S2: 20 back-to-back queries   | Send 20 messages rapidly                  | iPhone 12 | No jetsam, no OOM, all responses valid |
| S3: Long conversation         | 50-message conversation                   | iPhone 12 | Peak memory <700MB                     |
| S4: RAG-heavy query           | "What exercises did I do last week?"      | iPhone 12 | Response includes RAG data, <8s        |
| S5: Background/foreground     | Chat → Background 30s → Foreground → Chat | iPhone 12 | No crash, chat works                   |
| S6: Memory warning simulation | Simulate memory warning during inference  | iPhone 12 | Graceful degradation, no crash         |
| S7: Sustained inference       | 10-minute chat session                    | iPhone 12 | No memory growth trend                 |

### Instrumentation to Add

```swift
// os_signpost for inference phases
import os.signpost
let log = OSLog(subsystem: "com.evo.evotraining", category: "Inference")
os_signpost(.begin, log: log, name: "PromptAssembly")
// ... prompt assembly ...
os_signpost(.end, log: log, name: "PromptAssembly")

// Memory tracking
func logMemoryUsage(phase: String) {
    var info = mach_task_basic_info()
    var count = mach_msg_type_number_t(MemoryLayout<mach_task_basic_info>.size) / 4
    let result = withUnsafeMutablePointer(to: &info) {
        $0.withMemoryRebound(to: integer_t.self, capacity: 1) {
            task_info(mach_task_self_, task_flavor_t(MACH_TASK_BASIC_INFO), $0, &count)
        }
    }
    if result == KERN_SUCCESS {
        let usedMB = Double(info.resident_size) / 1_000_000
        print("[MEMORY] \(phase): \(String(format: "%.1f", usedMB))MB")
        CrashLogger.shared.log("[MEMORY] \(phase): \(String(format: "%.1f", usedMB))MB", level: "INFO")
    }
}
```

### Metrics to Track

- **Peak resident memory** (per scenario)
- **Dirty memory** (heap, not mmap'd)
- **Jetsam events** (via MetricKit `MXAppExitMetric`)
- **Inference latency** (P50, P95)
- **Time to first token**
- **Memory warning count**
- **RAG retrieval latency**

### Acceptance Criteria

| Metric                      | Tier L Target        | Tier H Target |
| --------------------------- | -------------------- | ------------- |
| Peak resident memory        | <700MB               | <2GB          |
| Jetsam rate                 | 0% in test scenarios | 0%            |
| Inference P95 latency       | <8s                  | <3s           |
| Time to first token         | <3s                  | <1s           |
| Memory warnings during chat | <2 per session       | 0             |

---

## 9. Two-Week Implementation Sequence

### Week 1: Foundation & Quick Wins

**Day 1-2: Instrumentation**

- [ ] Add `logMemoryUsage()` calls at key phases
- [ ] Add os_signpost markers for inference pipeline
- [ ] Baseline memory measurements on iPhone 12 and iPhone 17 PM
- [ ] Document current peak memory per scenario
- **Checkpoint**: Baseline metrics captured

**Day 3: Verify Existing Tier Config**

- [ ] Audit n_ctx, n_batch, n_threads settings per tier
- [ ] Verify CHUNK_CONFIG logs show expected values
- [ ] Fix any config inconsistencies
- **Checkpoint**: Config verified, no regressions

**Day 4-5: Quick Wins (A1-A5)**

- [ ] A3: Implement chat history limit (5 messages on Tier L)
- [ ] A4: Evict KV cache on background
- [ ] A5: Unload TFLite embedding model after RAG
- [ ] Feature flags for all changes
- **Checkpoint**: Quick wins deployed behind flags

**Day 6-7: Testing Quick Wins**

- [ ] Run all test scenarios on iPhone 12
- [ ] Measure memory reduction
- [ ] Verify no regressions in chat quality
- [ ] A/B test with flags on/off
- **Checkpoint**: Quick wins validated, flags enabled for Tier L

### Week 2: Medium Effort & Polish

**Day 8-9: RAG Optimizations (B1, B5, C1)**

- [ ] B1: Lazy-load RAG index
- [ ] B5: Reduce top-k to 3 on Tier L
- [ ] C1: Implement LRU snippet cache
- **Checkpoint**: RAG optimizations deployed behind flags

**Day 10-11: Prompt Optimizations (B3, B4)**

- [ ] B3: Streaming prompt assembly (if feasible)
- [ ] B4: Memory capsule implementation (rule-based)
- **Checkpoint**: Prompt optimizations deployed

**Day 12: Memory Pressure Observer (C4)**

- [ ] Implement `didReceiveMemoryWarning` handler
- [ ] Trigger cleanup: evict caches, clear image cache, trim history
- **Checkpoint**: Memory pressure handling deployed

**Day 13: Full Integration Testing**

- [ ] Run all scenarios on iPhone 12 with all flags enabled
- [ ] Run all scenarios on iPhone 17 PM (verify no regression)
- [ ] Stress test: 100 messages, background/foreground cycles
- **Checkpoint**: All tests pass

**Day 14: Documentation & Rollout**

- [ ] Document all feature flags and their effects
- [ ] Create rollout plan: Tier L first, then Tier H
- [ ] Enable flags in production for Tier L
- [ ] Monitor MetricKit for jetsam events
- **Checkpoint**: Production rollout complete for Tier L

### Rollback Plan

Each feature flag can be disabled remotely via:

1. UserDefaults key (for testing)
2. Remote config (for production)
3. App update (worst case)

### Success Criteria for Week 2 End

- [ ] iPhone 12 peak memory <700MB in all scenarios
- [ ] Zero jetsam events in test scenarios
- [ ] Inference latency within 20% of baseline
- [ ] No user-reported regressions in chat quality

---

## Summary

This plan prioritizes **reducing peak memory** through:

1. **Config enforcement** (n_ctx=512, n_batch=32 on Tier L)
2. **Lazy loading** (RAG index, embedding model)
3. **Aggressive eviction** (KV cache on background, memory warning handler)
4. **Prompt compression** (history trimming, memory capsules, reduced top-k)
5. **Instrumentation** (measure before/after, catch regressions)

All changes are **feature-flagged**, **tier-gated**, and **reversible**. The 2-week timeline allows for safe, incremental rollout with validation at each checkpoint.

## Related
