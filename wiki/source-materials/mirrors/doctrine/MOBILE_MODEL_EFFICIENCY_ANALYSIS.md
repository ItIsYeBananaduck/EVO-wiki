---
title: MOBILE_MODEL_EFFICIENCY_ANALYSIS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/MOBILE_MODEL_EFFICIENCY_ANALYSIS.md"]
updated: 2026-07-24
---

# Mobile Model Efficiency Analysis: Phi-4-mini Q4_K_M

## Executive Summary

**Phi-4-mini Q4_K_M is too heavy for low-end and mid-range mobile devices.** It works on high-end devices (iPhone 15 Pro+) but with high energy consumption. On iPhone 12-class devices, it causes thermal throttling, UI freezes, and potential system kills.

**Recommendation**: Adopt a **tiered model strategy** — serve a smaller model (1.5B-class) on low/mid devices and keep Phi-4-mini for high-end only.

---

## 1. Phi-4-mini Architecture Facts

| Property         | Value                                |
| ---------------- | ------------------------------------ |
| Parameters       | 3.8B                                 |
| Layers           | 32 Transformer layers                |
| Hidden size      | 3,072                                |
| Attention heads  | 32 (GQA: 8 KV heads)                 |
| Vocabulary       | 200,064 tokens (o200k_base tiktoken) |
| Training context | 128K tokens                          |
| Quantization     | Q4_K_M (4-bit K-quant)               |
| GGUF file size   | **2.37 GB**                          |
| ENF LoRA adapter | **88 MB**                            |

## 2. Memory Footprint Breakdown

### Model Weights (on disk → in RAM)

- Q4_K_M weights: **~2.4 GB** (mmap'd, but active pages still consume RAM)
- With `use_mmap=true`, iOS will page in/out, but hot layers stay resident

### KV Cache (the real killer)

Formula: `2 × n_layers × n_kv_heads × head_dim × n_ctx × bytes_per_element`

For Phi-4-mini with GQA (8 KV heads, head_dim = 3072/32 = 96):

```
KV cache per token = 2 × 32 × 8 × 96 × 2 bytes (FP16) = 98,304 bytes ≈ 96 KB/token
```

| Context Size  | KV Cache (FP16) | KV Cache (Q4_0) |
| ------------- | --------------- | --------------- |
| 2,048 tokens  | **192 MB**      | ~48 MB          |
| 4,096 tokens  | **384 MB**      | ~96 MB          |
| 8,192 tokens  | **768 MB**      | ~192 MB         |
| 16,384 tokens | **1,536 MB**    | ~384 MB         |

### Compute Buffer

- Phi-4-mini's 200K vocabulary requires a large embedding/output projection buffer
- Compute buffer: **~200-500 MB** depending on batch size and context

### Total Memory Budget

| Component                 | Low Estimate | High Estimate |
| ------------------------- | ------------ | ------------- |
| Model weights (hot pages) | 1.5 GB       | 2.4 GB        |
| KV cache (2K ctx, FP16)   | 192 MB       | 192 MB        |
| Compute buffer            | 200 MB       | 500 MB        |
| ENF LoRA adapter          | 88 MB        | 88 MB         |
| OS/App overhead           | 500 MB       | 800 MB        |
| **TOTAL**                 | **~2.5 GB**  | **~4.0 GB**   |

## 3. Device Capability Matrix

### iOS Device RAM and Available Memory

| Device              | Total RAM | Available for App | GPU Memory BW |
| ------------------- | --------- | ----------------- | ------------- |
| iPhone 12 / 12 mini | 4 GB      | ~2.0-2.5 GB       | 34 GB/s       |
| iPhone 13 / 13 mini | 4 GB      | ~2.0-2.5 GB       | 34 GB/s       |
| iPhone 14           | 6 GB      | ~3.0-3.5 GB       | 34 GB/s       |
| iPhone 14 Pro       | 6 GB      | ~3.0-3.5 GB       | 34 GB/s       |
| iPhone 15           | 6 GB      | ~3.0-3.5 GB       | 34 GB/s       |
| iPhone 15 Pro       | 8 GB      | ~4.5-5.5 GB       | 34 GB/s       |
| iPhone 16 Pro       | 8 GB      | ~4.5-5.5 GB       | 34 GB/s       |
| iPhone 16 Pro Max   | 12 GB     | ~7.0-8.0 GB       | 34 GB/s       |

### Verdict by Device Tier

| Tier      | Devices               | RAM   | Phi-4-mini Feasible? | Issue                                                                        |
| --------- | --------------------- | ----- | -------------------- | ---------------------------------------------------------------------------- |
| **Low**   | iPhone 12, 13         | 4 GB  | **NO**               | Model alone exceeds available RAM. System kill guaranteed.                   |
| **Mid**   | iPhone 14, 15         | 6 GB  | **Barely**           | Runs but leaves <500MB headroom. Thermal throttling, UI jank, battery drain. |
| **High**  | iPhone 15 Pro, 16 Pro | 8 GB  | **Yes, with care**   | Works but energy-intensive. Context must stay ≤4K.                           |
| **Ultra** | iPhone 16 Pro Max     | 12 GB | **Yes**              | Comfortable. Can use 8K context.                                             |

## 4. Performance Estimates

### Token Generation Speed (decode, tokens/sec)

Mobile memory bandwidth is ~34 GB/s. Decode is memory-bound.

```
Theoretical max TPS = memory_bandwidth / model_size_in_memory
                    = 34 GB/s / 2.4 GB ≈ 14 tokens/sec (theoretical ceiling)
```

Real-world with overhead, KV cache reads, and Metal scheduling:

| Device                   | Expected TPS         | User Experience                       |
| ------------------------ | -------------------- | ------------------------------------- |
| iPhone 12 (4GB)          | 2-5 t/s (if it runs) | Unusable — constant swapping, freezes |
| iPhone 14 (6GB)          | 5-8 t/s              | Sluggish — noticeable delays          |
| iPhone 15 Pro (8GB)      | 8-12 t/s             | Acceptable — slight lag               |
| iPhone 16 Pro Max (12GB) | 10-14 t/s            | Good — responsive                     |

### Prompt Processing Speed (prefill)

Prefill is compute-bound, not memory-bound. With batch size 64:

| Device        | Expected PP speed |
| ------------- | ----------------- |
| iPhone 12     | 20-40 tokens/sec  |
| iPhone 15 Pro | 60-100 tokens/sec |

A 500-token system prompt takes **5-25 seconds** to process on iPhone 12.

## 5. The 200K Vocabulary Problem

Phi-4-mini's 200K vocabulary is **4-5x larger** than typical models (Llama: 32K, Phi-3: 32K). This causes:

1. **Larger embedding table**: 200K × 3072 × 2 bytes = **1.17 GB** in FP16 (shared input/output, so ~600MB in Q4)
2. **Larger compute buffer**: The output projection (logits) requires a 200K-wide softmax every token
3. **Sampling crashes**: top_k + top_p crash with 200K vocab (llama.cpp #15961)
4. **Higher minimum context**: Needs n_ctx ≥ 2048 just for compute buffer allocation

This vocabulary size is designed for multilingual support, which Alice doesn't need for a fitness app.

## 6. Energy Consumption Analysis

### Why Energy Is High Even on 12GB Devices

1. **Full GPU offload (99 layers → now 80)**: GPU runs at max frequency continuously
2. **Large KV cache**: Every decode step reads/writes 192MB+ of KV cache
3. **200K vocab softmax**: Every token requires scoring 200K candidates
4. **No idle periods**: Autoregressive decode keeps GPU busy with no gaps
5. **Thermal throttling**: Sustained GPU load triggers thermal management, reducing clocks

### Estimated Power Draw

| Model Size           | Estimated Power (sustained) |
| -------------------- | --------------------------- |
| 1.5B Q4              | ~2-3W                       |
| 3.8B Q4 (Phi-4-mini) | **~5-7W**                   |
| 7B Q4                | ~8-10W                      |

At 5-7W, a 30-second generation drains ~0.05% battery per query. But the real issue is **thermal throttling** — the phone gets hot, clocks drop, and everything slows down.

---

## 7. Proposed Solution: Tiered Model Strategy

### Option A: Single Smaller Model (Simplest)

Replace Phi-4-mini with a **1.5B-class model** that works on ALL devices:

| Model                     | Params | Q4_K_M Size | Vocab | Quality                            |
| ------------------------- | ------ | ----------- | ----- | ---------------------------------- |
| **Qwen2.5-1.5B-Instruct** | 1.5B   | ~900 MB     | 151K  | Strong for size, good tool calling |
| **Phi-3.5-mini**          | 3.8B   | ~2.2 GB     | 32K   | Smaller vocab, proven on mobile    |
| **SmolLM2-1.7B-Instruct** | 1.7B   | ~1.0 GB     | 49K   | Optimized for edge                 |
| **Gemma-3-1B**            | 1B     | ~600 MB     | 262K  | Google's edge model                |
| **Llama-3.2-1B-Instruct** | 1.2B   | ~700 MB     | 128K  | Meta's mobile model                |

**Recommendation: Qwen2.5-1.5B-Instruct**

- 900MB file size (vs 2.4GB) — 62% smaller
- ~50MB KV cache at 2K context (vs 192MB) — 74% less
- ~8-15 t/s on iPhone 12 (vs 2-5 t/s)
- Native tool calling support
- Good instruction following

### Option B: Tiered Model System (Best UX)

Serve different models based on device capability:

```
Low-end  (4GB):  Qwen2.5-1.5B Q4_K_M  (~900 MB)
Mid-end  (6GB):  Qwen2.5-1.5B Q4_K_M  (~900 MB)
High-end (8GB+): Phi-4-mini Q4_K_M    (~2.4 GB)
```

**Implementation**:

1. Add model selection logic in `AliceAssetDownloadManager`
2. Download the appropriate model based on `ProcessInfo.processInfo.physicalMemory`
3. Both models use the same llama.cpp inference path
4. LoRA adapters would need to be retrained for the smaller model

### Option C: Aggressive Quantization of Phi-4-mini (Quick Win)

Re-quantize Phi-4-mini to smaller formats:

| Quantization     | File Size | Quality Loss | Memory Savings |
| ---------------- | --------- | ------------ | -------------- |
| Q4_K_M (current) | 2.37 GB   | Baseline     | Baseline       |
| **IQ4_XS**       | ~2.0 GB   | Minimal      | ~15%           |
| **Q3_K_M**       | ~1.9 GB   | Noticeable   | ~20%           |
| **IQ3_XXS**      | ~1.6 GB   | Significant  | ~33%           |
| **Q2_K**         | ~1.4 GB   | Major        | ~41%           |

**Problem**: Even at Q2_K, the model is still 1.4GB + KV cache + compute buffer = ~2.5GB total. This doesn't solve the iPhone 12 problem.

### Option D: KV Cache Quantization (Complementary)

llama.cpp supports quantized KV cache (`-ctk q4_0 -ctv q4_0`):

| KV Cache Type  | Memory at 2K ctx | Memory at 4K ctx |
| -------------- | ---------------- | ---------------- |
| FP16 (current) | 192 MB           | 384 MB           |
| **Q8_0**       | 96 MB            | 192 MB           |
| **Q4_0**       | 48 MB            | 96 MB            |

This is a **quick win** that can be combined with any option above.

---

## 8. Recommended Action Plan

### Phase 1: Quick Wins (This Week)

1. **Enable KV cache quantization** (Q8_0) — saves 50% KV memory
2. **Reduce context limits** for low-end devices (already partially done)
3. **Add device memory gate** — refuse to load Phi-4-mini on <6GB devices
4. **Show clear UX** — "Alice requires iPhone 14 or newer" for current model

### Phase 2: Tiered Model (Next Sprint)

1. **Quantize Qwen2.5-1.5B-Instruct to Q4_K_M GGUF**
2. **Upload to R2 storage** alongside Phi-4-mini
3. **Add model selection logic** in download manager
4. **Test tool calling** with smaller model
5. **Retrain ENF LoRA** for smaller model (or skip ENF on small model)

### Phase 3: Optimization (Following Sprint)

1. **Speculative decoding** — use 0.5B draft model for 2-3x speedup
2. **KV cache eviction** — StreamingLLM-style attention sinks for longer conversations
3. **Batch scheduling** — burst inference then sleep for better thermals

---

## 9. Summary Table

| Metric                | Current (Phi-4-mini Q4_K_M) | After Phase 1    | After Phase 2 (Low-end) |
| --------------------- | --------------------------- | ---------------- | ----------------------- |
| Model size            | 2.37 GB                     | 2.37 GB          | ~900 MB                 |
| KV cache (2K)         | 192 MB                      | **96 MB** (Q8_0) | ~25 MB                  |
| Total memory          | ~3.5 GB                     | ~3.3 GB          | **~1.5 GB**             |
| iPhone 12 support     | **NO**                      | **NO**           | **YES**                 |
| iPhone 14 support     | Barely                      | Better           | **YES**                 |
| iPhone 15 Pro support | Yes                         | Yes              | Yes                     |
| TPS (iPhone 12)       | 2-5 (crashes)               | N/A              | **8-15**                |
| TPS (iPhone 15 Pro)   | 8-12                        | 8-12             | 15-25                   |
| Energy (sustained)    | ~5-7W                       | ~4-6W            | **~2-3W**               |
| Init time             | 20-30s                      | 15-20s           | **5-8s**                |

---

_Analysis date: February 11, 2026_
_Author: Cascade AI Analysis_

## Related
