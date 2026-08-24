---
title: INFERENCE_CONFIG_REFACTOR_PLAN
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/INFERENCE_CONFIG_REFACTOR_PLAN.md
updated: 2026-07-24
---

# Inference Config Refactor Plan

## Problem Statement

- Metal-safe guards applied n_batch=64, context created with n_batch=64, but generation starts with n_batch=512
- Context created with n_ctx=2048 but generation start prints n_ctx=8192
- Prompt processing: 2869 tokens chunked into 90 chunks of 32 tokens (too many llama_decode calls)

## Solution: Single Source of Truth InferenceConfig

### 1. InferenceConfig Struct (COMPLETED)

- Created `InferenceConfig` struct with all inference parameters
- Includes: n_ctx, n_batch, n_gpu_layers, useMetal, flashAttn, chunkSize, n_threads
- Metal-safe guards built into `create()` method
- Adaptive sizing via `createAdaptive()` method

### 2. Context Creation Refactor (IN PROGRESS)

- Replace fixed `contextSize` and `batchSize` with `InferenceConfig`
- Create context using `inferenceConfig.n_ctx` and `inferenceConfig.n_batch`
- Store `inferenceConfig` instance for use in generation

### 3. Generation Refactor (PENDING)

- Use `inferenceConfig.chunkSize` for prompt ingestion (not hardcoded 32)
- Assert batch size matches config before decode
- Use `inferenceConfig.n_batch` everywhere, never hardcoded values

### 4. Adaptive Context Sizing (PENDING)

- Calculate required tokens before generation
- Create/update `InferenceConfig` with adaptive n_ctx
- Recreate context if n_ctx changes

### 5. Prompt Budgeting (PENDING)

- Implement token budgeter that limits prompt sections
- Only include tools/capmap/skills when needed
- Truncate memoryBrief to fit budget

## Implementation Steps

1. ✅ Create InferenceConfig struct
2. ⏳ Update loadModel() to create InferenceConfig and use it for context creation
3. ⏳ Update generate() to use InferenceConfig for chunk sizing
4. ⏳ Add adaptive context sizing logic
5. ⏳ Implement prompt budgeting
6. ⏳ Add assertions to catch mismatches in debug builds

## Related

^[source-materials/mirrors/doctrine/INFERENCE_CONFIG_REFACTOR_PLAN.md]
