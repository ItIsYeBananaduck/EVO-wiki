---
title: ENF_LORA_TRAINING_SPEC
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/ENF_LORA_TRAINING_SPEC.md
updated: 2026-07-24
---

# ENF LoRA Training Specification

## Overview

The Enforcer LoRA (ENF) is a behavioral policy layer that acts as the highest-authority guardrail system for Alice. It enforces:

- Guardrails and safety constraints
- Role-based access control
- Output contract compliance
- Prevention of tool-claim hallucinations

## Training Dataset Composition

### 40% Allowed Responses

- Valid fitness coaching responses that follow all constraints
- Proper output format with `<policy>`, `<actions>`, `<answer>` tags
- Actions that respect `requiresPro` flags and capability tiers
- Responses that stay within fitness domain (unless admin role)

### 40% Refusal Responses

- Clear refusals when capabilities don't allow actions
- Polite redirects for non-fitness topics (non-admin users)
- Safety refusals for dangerous workout suggestions
- Format compliance refusals (when output contract is violated)

### 20% Capability Mismatch

- Responses that claim actions when `agenticEnabled=false`
- Tool claims (email, purchase, schedule) without permission
- Role violations (user trying to access trainer-only features)
- Tier violations (free tier trying to use pro features)

## Adversarial Jailbreaks

Include adversarial examples:

- Prompt injection attempts
- Instruction leakage attempts
- Format manipulation attempts
- Role/tier escalation attempts

## Input Format

Training examples should include:

- System prompt with capabilities header
- Domain context (workout, nutrition, recovery, planning)
- User message
- Expected response (with policy/actions/answer structure)

## Evaluation Criteria

1. **No Instruction Leak**: Model should never reveal system instructions or format requirements
2. **No Tool-Claim Hallucinations**: Model should never claim actions it cannot perform
3. **Correct Action Gating**: Actions with `requiresPro=true` only when `agenticEnabled=true`
4. **Output Contract Compliance**: Always includes `<policy>`, `<actions>`, `<answer>` tags
5. **Role Enforcement**: Non-admin users redirected from non-fitness topics
6. **Safety First**: Dangerous suggestions are refused with safety explanations

## Export Format

- **Model**: `enforcer_lora.gguf` (GGUF format for llama.cpp)
- **Metadata**: `enforcer_lora.meta.json` with:
  ```json
  {
    "version": "YYYYMMDD.N",
    "checksum": "sha256...",
    "trainedOnGuardrailVersion": "X.Y",
    "baseModelId": "alice-phi3-q4",
    "rank": 8,
    "steps": 2000
  }
  ```

## Training Parameters

- **Rank**: 8 (tunable, start with 8)
- **Alpha**: 16 (rank \* 2)
- **Steps**: 2000 (adjust based on dataset size)
- **Learning Rate**: 1e-4 (standard for LoRA)
- **Batch Size**: 4-8 (depending on GPU memory)

## Base Model

- **Model**: `alice-phi3-q4.gguf` (Phi-3 3.8B Q4_K_M quantization)
- **Format**: GGUF
- **Context**: 2048 tokens

## Deployment

1. Upload `enforcer_lora.gguf` to R2: `alice-assets/adapters/enforcer/enforcer_lora.gguf`
2. Upload `enforcer_lora.meta.json` alongside adapter
3. App will download and apply automatically on next inference
4. ENF is always applied first in adapter stack (highest priority)

## Related

^[source-materials/mirrors/doctrine/ENF_LORA_TRAINING_SPEC.md]
