---
title: CAPABILITY_MAP_REMOVED
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/CAPABILITY_MAP_REMOVED.md
updated: 2026-07-24
---

# Capability Map Removed - Card-Based Approach Only

## Change Summary

**Removed**: Full capability map loading (`formatCapabilityMap`, `CAPABILITY_MAP` section)
**Kept**: Card-based tool loading (only specific tools needed)

## Why This Matters

The capability map was **2000+ tokens** of verbose action definitions. The card-based approach loads only the specific tools needed (typically 100-300 tokens).

## Changes Made

1. **Removed capability map formatting** (line 2595-2620):
   - `formatCapabilityMap()` call removed
   - `capabilityMapSection` always empty
   - `capabilityMapInstructions` always empty

2. **Updated system prompt building** (line 2688-2702):
   - Removed `capabilityMapSection` and `capabilityMapInstructions` from prompt
   - Only includes tool cards when needed

## Token Savings

- **Before**: Capability map = ~2000 tokens + tool cards = ~2300 tokens
- **After**: Tool cards only = ~100-300 tokens
- **Savings**: ~2000 tokens per request (when tools are needed)

## Card-Based Tool Loading

The system now uses:

- `detectNeededTools()` - Detects which specific tools are needed
- `generateToolDefinitionsJSON(requestedTools:)` - Loads only those tools
- No full capability map - saves massive tokens

## Impact

- **System tokens**: Reduced by ~2000 tokens when tools are needed
- **Total prompt**: Much smaller for tool-based queries
- **Prefill time**: Faster due to smaller prompt

## Related

^[source-materials/mirrors/doctrine/CAPABILITY_MAP_REMOVED.md]
