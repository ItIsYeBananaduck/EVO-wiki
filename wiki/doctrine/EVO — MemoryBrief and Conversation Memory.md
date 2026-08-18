---
title: EVO — MemoryBrief and Conversation Memory
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVO — MemoryBrief and Conversation Memory.md"]
updated: 2026-07-24
---

# EVO — MemoryBrief and Conversation Memory

## Purpose

Define the current MemoryBrief and conversation-memory path used by Alice inference. This note describes only behavior verified against current code.

## Storage Model

`ConversationMemoryManager` is the local-first memory store for Alice conversation context.

Verified properties:

- memory files live under `AliceAssets/memory/<appId>/<userId>/`
- the primary store is `memories.jsonl`
- writes are append-only for normal filesystem storage
- memory types are `semantic`, `episodic`, and `procedural`
- memory items may include tags, mode, importance, artifact type, domain, and correction metadata
- the store prunes after `maxMemories = 2000`
- vector index files live beside the JSONL store when embeddings are available

## Retrieval

`buildMemoryBrief()` is a backward-compatible wrapper around `buildCognitionBrief()`.

Current retrieval behavior:

- tokenizes the user query and removes common stopwords
- loads local memories
- filters by domain when a domain is provided
- filters by artifact type when artifact scope is provided
- uses hybrid retrieval when embeddings and the vector index are ready
- falls back to keyword-only scoring when embeddings are not available
- over-fetches vector results so domain and artifact filters can still keep enough in-scope results
- skips stale memories older than the configured age gate unless vector similarity is strong enough
- applies a stale-memory score penalty when old memory is allowed through

Hybrid scoring combines vector and keyword signals:

- vector score weight: 70%
- normalized keyword score weight: 30%

Keyword fallback scoring includes:

- query keyword overlap
- importance
- recency
- tag matches
- mode match

## Selection

The brief selects memories by type, with device-tier-sensitive caps:

- low memory tier: 3 semantic, 2 procedural, 2 episodic
- mid memory tier: 4 semantic, 3 procedural, 3 episodic
- high/default tier: 6 semantic, 4 procedural, 4 episodic

The default maximum is 40 lines unless a caller passes a different limit.

Correction-aware recall keeps the latest correction for a source memory and skips the corrected source item when both appear in the selected set.

## Formatting

The current formatted block is a cognition brief:

```text
[COGNITION BRIEF]
• [FACT] semantic memory text
• [TRIED] procedural memory text
• [HAPPENED] episodic memory text
[/COGNITION BRIEF]
```

Assistant-like memories are filtered before injection to reduce response regurgitation.

## AliceBrainService Injection

`AliceBrainService.generate()` builds the brief before the native generation call:

1. lazy-loads `ConversationMemoryManager`
2. calls `buildMemoryBrief()` with the user message, request domain as mode, request domain as domain, and `maxLines: 40`
3. optionally appends pattern context, nutrition-performance context, recent chat context, training cognition, and journal inference context
4. passes the combined `memoryBrief` through the platform channel as `memoryBrief`

The combined context uses labeled sections such as `[MEMORY CONTEXT]`, `[PATTERN CONTEXT]`, and `[JOURNAL ENTRY CONTEXT]`.

## Prompt Placement

`PromptBuilder.swift` accepts `memoryBrief`.

In tool mode, the prompt builder appends `CAPABILITIES_JSON`, optional policy overlay, then memory context before conversation history and before the tools block. In base mode, memory context is also inserted into the prompt as background context and budgeted against the prompt limit.

Memory context is explicitly framed as background only and not something to quote or repeat verbatim.

## Native Prompt Budgeting

The native generation path also budgets memory context:

- memory injection can be disabled by memory router mode
- memory context is truncated when it exceeds prompt budget
- constraint overlays may be prepended to the memory context

## Exclusions

This note does not document removed adapter behavior, removed training pipelines, or old repair-pass behavior from historical implementation notes.

## Related

^[{src_rel}]
