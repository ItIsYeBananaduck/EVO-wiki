---
title: CAPABILITY_MAP_COMPLETE
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CAPABILITY_MAP_COMPLETE.md"]
updated: 2026-07-24
---

# Capability Map - Complete Reference Guide

## ✅ What the Map Contains

The `alice_capability_map.json` tells Alice:

### 1. **What Functions Exist** ✅

- **17 Agentic Actions**: All functions Alice can perform
- **9 Automatic Actions**: System-triggered functions with specific triggers
- **Total: 26 functions** - Complete list, nothing missing

### 2. **What Each Function Does** ✅

- **`whatItDoes`** field for every function
- Explains: what the function does, what data it modifies, what the user experiences, how it works, why it matters
- Example: `update_workout_plan` explains it creates OR modifies plans, stores data, affects future workouts

### 3. **When to Use Each Function** ✅

- **`whenToUse`** array for every function
- Lists specific scenarios: user requests, system triggers, conditions
- Example: `adjust_live_rest` lists: user requests, strain >85%, HR >88%, RPE ≥9

### 4. **How to Use Each Function** ✅

- **`howToUse`** section for every function
- Step-by-step instructions:
  - Step 1: Check access rules
  - Step 2: Extract/identify data needed
  - Step 3: Construct payload
  - Step 4: Format action in <actions> tag
  - Step 5: Generate <answer>
- Includes format examples showing exact JSON structure

### 5. **Access Rules** ✅

- **`access`** object for every function
- Shows: requiresPro, requiresAdmin, requiresConfirmation, autoExecute, agenticEnabled requirements
- Domain constraints (live_workout, marketplace restrictions)

### 6. **Payload Structure** ✅

- **`payload`** object for every function
- Shows exactly what fields to include
- Data types, optional vs required, examples

### 7. **Examples** ✅

- **`examples`** array for every function
- Real usage scenarios showing input → action mapping

## Complete Coverage

### Agentic Actions (17) - All Have:

- ✅ `whatItDoes` - What the function does
- ✅ `whenToUse` - When to use it
- ✅ `howToUse` - Step-by-step instructions
- ✅ `access` - Access rules
- ✅ `payload` - Payload structure
- ✅ `examples` - Usage examples

### Automatic Actions (9) - All Have:

- ✅ `whatItDoes` - What the function does
- ✅ `trigger` - When it triggers
- ✅ `action` - Which agentic action it uses
- ✅ `autoExecute` - Execution rules
- ✅ `requiresPro` - Access requirements

## How Alice Uses It

When Alice needs to perform an agentic task:

1. **Consult the map** - Find the appropriate function
2. **Read `whatItDoes`** - Understand what it does
3. **Check `whenToUse`** - Confirm this is the right function
4. **Check `access`** - Verify she can use it (requiresPro, agenticEnabled)
5. **Follow `howToUse`** - Step-by-step instructions
6. **Construct payload** - Using `payload` structure
7. **Format action** - Using examples as reference
8. **Generate answer** - Explain what will happen

## Example Flow

**User**: "Create a push/pull/legs plan"

Alice consults map:

1. Function: `update_workout_plan` or `plan.create`
2. `whatItDoes`: Creates new workout plan from scratch
3. `whenToUse`: User says "create" → CREATE (omit planId)
4. `access`: requiresPro=true → check agenticEnabled ✅
5. `howToUse`: Step 1-6 instructions
6. Payload: `{"changes": {"schedule": {"type": "push_pull_legs", "daysPerWeek": 6}}}`
7. Format: `<actions>{"actions": [{"type": "update_workout_plan", ...}]}</actions>`
8. Answer: "I'll create a new push/pull/legs plan for you. This requires your confirmation."

## Benefits

1. **Explicit Knowledge** - Alice doesn't need to learn from training data alone
2. **Up-to-date** - Map can be updated without retraining
3. **Consistency** - All actions follow same structure
4. **Complete** - Every function documented
5. **Actionable** - Step-by-step instructions for each function

## Status

✅ **COMPLETE** - All 17 agentic actions have full documentation:

- What they do
- When to use them
- How to use them
- Access rules
- Payload structures
- Examples

The capability map is ready for Alice to reference!

## Related

^[{src_rel}]
