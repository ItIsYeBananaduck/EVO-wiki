---
title: ENF_MEMORY_IMPLEMENTATION_SUMMARY
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/ENF_MEMORY_IMPLEMENTATION_SUMMARY.md
updated: 2026-07-24
---

# ENF LoRA + RAG Memory Implementation Summary

## Overview

Implemented two features in EVOLoRA Mesh:

1. **ENF LoRA (Enforcer)**: Always-on behavioral policy layer
2. **RAG Memory (MemoryBrief)**: Conversation history retrieval and injection

## Implementation Details

### A) ENF LoRA (Enforcer)

#### Flutter/Dart Changes

1. **LoRAKind.ENF** added to `lora_kind.dart`
2. **LoRAAdapterManager** updated:
   - ENF adapter path: `adapters/enforcer/enforcer_lora.gguf`
   - R2 storage path: `alice-assets/adapters/enforcer/enforcer_lora.gguf`
   - Metadata support: `getEnforcerMetadataPath()`, `getEnforcerMetadata()`
   - Expected size: ~50MB

3. **MeshRouter** updated:
   - `getAdapterStackForNative()` always includes ENF first
   - ENF inserted at beginning of adapter stack for priority
   - Logging when ENF found/missing

#### iOS Swift Changes

1. **getEnforcerAdapterPath()** added:
   - Checks AppGroup, Documents/EVO/ModelStore, legacy paths
   - Same pattern as `getModelPath()`

2. **applyAdapterStack()** updated:
   - Always inserts ENF if not present
   - ENF applied first (highest priority)
   - Signature includes ENF for caching
   - Logging: adapter kinds, ENF applied status

3. **removeLoRAAdapter()** protection:
   - Refuses to remove ENF adapter
   - Error logged if attempted

4. **Diagnostics**:
   - ENF included in `getLoadedAdapters()`
   - Logs show ENF path, scale, applied status

### B) RAG Memory (MemoryBrief)

#### Flutter/Dart Changes

1. **ConversationMemoryManager** created:
   - Local-first storage: `AliceAssets/memory/<appId>/<userId>/`
   - JSONL format: `memories.jsonl` (append-only)
   - Memory types: semantic, episodic, procedural
   - Write policy: Only on "events" (preferences, bugs, decisions, config changes)
   - Pruning: Max 2000 memories, removes oldest low-importance

2. **buildMemoryBrief()** retrieval:
   - Keyword-based scoring (no embeddings yet)
   - Scoring: keywordOverlap*3 + importance*2 + recency + tagMatch + modeMatch
   - Selection: semantic (3), procedural (2), episodic (2), max 18 lines
   - Format: `[MEMORY BRIEF]` with `[FACT]`, `[TRIED]`, `[HAPPENED]` prefixes
   - Truncation: ~1200-1500 chars max

3. **AliceBrainService** updated:
   - Accepts `ConversationMemoryManager` in constructor
   - Builds `memoryBrief` before platform call
   - Passes `memoryBrief` to iOS via platform channel

#### iOS Swift Changes

1. **generate()** methods updated:
   - Added `memoryBrief: String?` parameter
   - Passed through call chain to `_processGeneration()`

2. **buildSystemPrompt()** updated:
   - Accepts `memoryBrief` parameter
   - Sanitizes angle brackets (prevents parsing breaks)
   - Injects after capabilities header, before core system
   - Logging: char count, line count

3. **PendingRequest** updated:
   - Includes `memoryBrief` field
   - All pending request processing updated

### C) Optional 2-Pass Repair

#### iOS Swift Changes

1. **detectViolations()** added:
   - Checks agentic actions when `agenticEnabled=false`
   - Detects impossible action claims in text
   - Checks output contract (missing answer)

2. **runRepairPass()** added:
   - Runs only when violations detected
   - Same adapters (including ENF)
   - Additional safety overlay in prompt
   - Small token limit (128) for fast repair
   - Synchronous (on generationQueue)

3. **Integration**:
   - Called after `parseStructuredResponse()`
   - If violations found, repair pass runs
   - Repair result replaces original if valid

### D) Logging & Diagnostics

#### Flutter

- Adapter stack: logs kinds in order (includes ENF)
- MemoryBrief: logs line count, top memory IDs/tags, retrieval time
- Memory store: logs size, pruning events

#### iOS

- applyAdapterStack: logs final adapter kinds, ENF included status
- buildSystemPrompt: logs MemoryBrief char/line count
- Repair pass: logs triggered status, reason
- ENF: logs path, scale, applied status

## Files Modified

### Flutter/Dart

- `flutter_app/lib/features/evolora_mesh/lora_kind.dart`
- `flutter_app/lib/features/alice/domain/lora_adapter_manager.dart`
- `flutter_app/lib/features/alice/domain/mesh_router.dart`
- `flutter_app/lib/features/alice/domain/alice_brain_service.dart`
- `flutter_app/lib/features/alice/domain/conversation_memory_manager.dart` (NEW)

### iOS Swift

- `flutter_app/ios/Runner/LlamaEngine.swift`
- `flutter_app/ios/Runner/AliceInferenceManager.swift`
- `flutter_app/ios/Runner/AppDelegate.swift`

## Testing Checklist

- [ ] ENF adapter downloaded and applied on every request
- [ ] ENF cannot be removed via `removeLoRAAdapter()`
- [ ] MemoryBrief generated and injected into prompts
- [ ] MemoryBrief sanitized (no angle bracket issues)
- [ ] Repair pass triggers on violations
- [ ] Repair pass only runs when violations detected (not always)
- [ ] Logging shows ENF and MemoryBrief status
- [ ] Existing U/T/GU/GT adapters still work

## Next Steps

1. Train ENF LoRA using training spec
2. Deploy ENF adapter to R2
3. Test end-to-end with real inference
4. Monitor repair pass frequency (should be rare)
5. Consider adding embeddings for better memory retrieval (future)

## Related

^[source-materials/mirrors/doctrine/ENF_MEMORY_IMPLEMENTATION_SUMMARY.md]
