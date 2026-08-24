---
title: EVO — Alice Long-Term Memory Recall Architecture
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVO — Alice Long-Term Memory Recall Architecture.md
updated: 2026-07-24
---

# EVO — Alice Long-Term Memory Recall Architecture
## Purpose

Define the intended architecture for Alice’s long-term memory recall system.

This system is meant to replicate the useful behavior pattern from OpenClaw-style memory recall while remaining aligned with EVO principles:

- local-first
- on-device by default
- small-context efficient
- semantically useful
- auditable
- separable from doctrine and workflow retrieval

## Core Goal

Allow Alice to remember meaningful prior context across sessions without requiring the full conversation history to stay in the live context window.

The system should:

- persist important conversational information
- retrieve only the most relevant memories for the current interaction
- support semantic recall instead of exact-match lookup
- keep prompt assembly lightweight and targeted

This is not a general document RAG system.

This is an Alice long-term memory recall system.

## What This System Is

Alice long-term memory recall should behave like:

- memory capture
- memory compression
- memory indexing
- semantic/hybrid recall
- top-K selection
- prompt injection

The intent is:

1. a conversation produces memory-worthy information
2. that information is distilled into durable memory records
3. those records are indexed for later retrieval
4. future conversations retrieve the most relevant records
5. only the best matching memories are inserted into active context

## What This System Is Not

This system is not:

- short-term chat history replay
- raw transcript stuffing
- doctrine retrieval
- notes search
- issue/workflow retrieval
- broad external knowledge retrieval

Keep these separate.

### Separation Boundaries

Used for:

- current session continuity
- recent turn awareness
- immediate response coherence

Used for:

- durable personal/contextual recall across sessions
- semantically relevant memory resurfacing
- compressed continuity

Used for:

- system behavior rules
- structured workflows
- project knowledge
- authoritative design references

These should not be merged into one memory bucket.

## Memory Lifecycle

### 1. Capture

The system observes conversation turns, summaries, user preferences, repeated facts, important decisions, and emotionally or functionally meaningful events.

### 2. Memory Candidate Filtering

Not every interaction becomes long-term memory.

A candidate memory should be stored only if it is likely to matter later.

Examples of memory-worthy content:

- durable preferences
- recurring goals
- important project decisions
- user-specific constraints
- emotionally meaningful personal context
- trainer / plan / authority relationships
- important app-behavior expectations

Examples of non-memory-worthy content:

- small talk
- ephemeral acknowledgements
- redundant short-term phrasing
- already-stored equivalent content

### 3. Compression / Distillation

Captured material should be converted into a compact memory record.

Do not store full raw transcript by default if a compact summary is sufficient.

Each memory record should preserve:

- meaning
- source context
- enough detail for useful recall
- traceability

### 4. Indexing

Memory records should be added to a local index.

This is where `VectorMemoryIndex` or equivalent fits.

The index should support:

- semantic similarity lookup
- efficient top-K retrieval
- optional metadata filtering
- optional lexical / keyword assist

### 5. Retrieval

When a new conversation or prompt arrives:

- build a query representation from the current prompt and recent context
- retrieve relevant memory candidates
- rank them
- select only the best few

### 6. Prompt Injection

Only the most relevant recalled memories should be injected into active prompt context.

Do not dump everything.

The retrieved memories should be:

- compact
- relevant
- clearly separated from current conversation context

## Intended Core Components

### VectorMemoryIndex

Likely responsibility:

- store memory vectors or vector-like semantic representations
- support semantic retrieval
- rank candidates by similarity
- return top-K matches
- optionally support hybrid scoring with metadata or lexical signals

Expected functions:

- add memory record to index
- update memory record
- query relevant memories
- return ranked candidates

### ConversationMemoryManager

Likely responsibility:

- decide what becomes long-term memory
- create memory records from conversation content
- call indexing logic
- manage retrieval requests
- prepare recalled memories for prompt insertion
- handle memory deduplication / consolidation / promotion rules

Expected functions:

- ingest candidate conversation memory
- summarize or compress memory
- persist memory metadata/content
- call vector index for recall
- return prompt-ready memory set

## Memory Record Shape

A long-term memory record should contain something like:

- memory ID
- user / scope association
- memory text or structured summary
- source type
- created timestamp
- updated timestamp
- semantic representation / vector reference
- tags / metadata
- importance or salience score
- optional recency weighting fields
- optional trace/reference back to source event

Optional metadata:

- project
- domain
- emotional salience
- preference
- trainer-managed / user-managed
- relationship / authority context
- confidence
- pinned / protected

## Retrieval Strategy

### Baseline

Use semantic retrieval as the main mechanism.

That means:

- current prompt
- recent conversational context
- current task framing

are used to retrieve relevant memories.

### Hybrid Retrieval

If following the OpenClaw pattern, use hybrid scoring where helpful:

- semantic similarity
- lexical/keyword matches
- metadata filters
- recency bias
- salience weighting

This improves recall quality and reduces false positives.

### Top-K

Keep retrieval narrow.

Only return a small number of memories, such as:

- top 3
- top 5
- top 10
- top 20 if working in a broader synthesis context

The exact K should depend on:

- model size
- available context window
- prompt complexity
- memory density

For small on-device models, smaller K is usually better.

## Prompt Assembly Rules

Retrieved memories should be injected as a distinct section.

Example structure:

- current user turn
- recent session context
- recalled long-term memories
- current system/domain instructions

Do not blur these together.

The model should be able to distinguish:

- what is happening now
- what is being recalled from long-term memory
- what is system guidance

## Local-First Requirements

This system should be on-device by default.

Requirements:

- memory storage local by default
- retrieval local by default
- no cloud dependency for basic recall
- should function offline
- should not require server-side RAG infrastructure

If sync exists later, it should be:

- optional
- controlled
- privacy-preserving
- clearly separated from core recall logic

## Quality Constraints

### Relevance

Only retrieve memories that materially help the current prompt.

### Compression

Memories should be compact enough for small models.

### Precision

Avoid flooding context with vaguely related memories.

### Deduplication

Do not repeatedly inject equivalent memories.

### Stability

Durable preferences and important facts should remain retrievable.

### Traceability

When needed, the system should be able to explain where a recalled memory came from.

## Failure Modes to Guard Against

### 1. Over-retrieval

Too many memories added to prompt.

Result:

- token waste
- lower relevance
- model distraction

### 2. Under-retrieval

No useful memory surfaced.

Result:

- Alice feels forgetful
- continuity breaks

### 3. Duplicate recall

Same memory returns repeatedly in slightly different forms.

Result:

- wasted context
- repetitive responses

### 4. Wrong-memory injection

Semantically adjacent but incorrect memory is recalled.

Result:

- hallucinated continuity
- user trust damage

### 5. Raw transcript bloat

Too much raw text stored or injected.

Result:

- poor compression
- context inefficiency

## Recommended Operating Principle

The system should optimize for:

minimal, relevant, semantically useful recall

Not:

- maximal archival replay

This is especially important for on-device models with limited context windows.

## Expected Relationship to Alice

Long-term memory recall should make Alice feel:

- continuous
- personally aware
- contextually grounded
- efficient

without making her feel:

- bloated
- over-persistent
- invasive
- server-dependent

## Implementation Intent

If the current repo already contains:

- `vector_memory_index.dart`
- `conversation_memory_manager.dart`

then these should be treated as the likely core of this system.

Audit focus should be:

- what is actually stored
- how indexing works
- whether retrieval is semantic or hybrid
- what top-K policy is used
- how recalled memories are injected into prompt context
- whether the system is fully local-first

## Open Questions

These should be answered during implementation audit:

6. What qualifies a conversation item as memory-worthy?
7. Is memory stored as raw transcript, distilled summary, or both?
8. Are vectors true embeddings or heuristic stand-ins?
9. Is retrieval purely semantic or hybrid?
10. What top-K is currently used?
11. How are duplicates handled?
12. How does memory injection interact with short-term conversation context?
13. Is the system already live, partially wired, or only scaffolded?

## Intended Outcome

Alice should be able to:

- remember important user context across sessions
- recall semantically relevant memories on-device
- stay efficient in small context windows
- preserve continuity without relying on cloud-side memory infrastructure

This is the target architecture for EVO long-term memory recall.

## Related

^[source-materials/mirrors/doctrine/EVO — Alice Long-Term Memory Recall Architecture.md]
