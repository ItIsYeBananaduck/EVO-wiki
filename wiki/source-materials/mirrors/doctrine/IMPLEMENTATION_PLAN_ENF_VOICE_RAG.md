---
title: IMPLEMENTATION_PLAN_ENF_VOICE_RAG
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-deprecated/IMPLEMENTATION_PLAN_ENF_VOICE_RAG.md"]
updated: 2026-07-24
---

# Implementation Plan: ENF/VOICE LoRA Handoff Mesh + RAG MemoryBrief

## Overview

Implement deterministic ENF/VOICE adapter stacking, RAG memory injection, ENF strict mode repair, and runtime action gating in iOS LlamaEngine.swift.

## Step-by-Step Plan

### Phase 1: Configuration & Adapter Stack Management

1. Add `MeshConfig` struct for ENF/VOICE settings
2. Enhance `computeFinalAdapterStack()` with explicit slots
3. Update adapter caching signature to include ENF/VOICE presence
4. Ensure proper adapter cleanup (no memory leaks)

### Phase 2: RAG MemoryBrief Injection

1. Enhance `buildMemoryOverlay()` with better sanitization
2. Strip dangerous tags from memoryBrief (prevent injection)
3. Add size/token limits (800-1200 chars)
4. Ensure memoryBrief never leaks into system instructions

### Phase 3: ENF Strict Mode & Repair

1. Add hard-coded repair function (no second model pass)
2. Detect violations: missing tags, invalid JSON, illegal actions
3. Generate safe fallback response
4. Log repair events for monitoring

### Phase 4: Runtime Action Gating

1. Add `applyHardGatingToActions()` function
2. Filter `requiresPro=true` actions when `agenticEnabled=false`
3. Enforce domain constraints (live_workout brevity)
4. Set actions to `none` if all filtered

### Phase 5: Method Channel Interface

1. Add `setMeshConfig` method channel handler
2. Update AppDelegate to handle config updates
3. Make config persistent (UserDefaults or file)

### Phase 6: Training Prep Scripts

1. Create ENF dataset generator (JSONL format)
2. Create VOICE dataset generator (JSONL format)
3. Add dataset validation scripts
4. Include M4 Mac training recommendations

### Phase 7: Evaluation & Metrics

1. Add metrics logging (repair rate, illegal action rate, leakage rate)
2. Create evaluation checklist
3. Add diagnostic endpoints for monitoring

## Related
