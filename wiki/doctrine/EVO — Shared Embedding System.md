---
title: EVO — Shared Embedding System
type: concept
tags: [evo, system]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# EVO — Shared Embedding System

> NOTE: This is a canonical doctrine note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

> RULE: All `related` entries must use Obsidian wiki link format.

---

## Purpose

Define how semantic memory is stored, indexed, and retrieved across all EVO apps using a shared embedding space.

The embedding system enables:

- cross-app context discovery  
- efficient memory retrieval  
- seamless conversation continuity  
- domain-aware knowledge access  

---

## Core Principle

Embeddings provide **shared discoverability**, not shared ownership.

All domains contribute to a shared semantic index, but access to underlying data is governed by domain rules.

---

## Definitions

**Embedding**  
A vector representation of content (message, note, log, journal) used for semantic search.

**Shared Embedding Space**  
A global index of embeddings generated from all EVO apps.

**Source Record**  
The original data object associated with an embedding (log, note, journal, etc.).

**Retrieval Context**  
The set of embeddings selected as relevant to a query.

---

## System Structure

Content Created (chat, note, log, journal)  
→ Embedding Generated  
→ Stored in Shared Embedding Space  
→ Indexed with metadata  
→ Retrieved based on semantic similarity  

---

## Embedding Sources

Embeddings may be generated from:

- chat messages  
- journal entries  
- logs (summarized or structured)  
- living notes  
- workflow decisions (Connect)  

---

## Metadata (Critical)

Every embedding must include:

- source app (Training, Mind, Learn, Connect)  
- content type (chat, log, journal, note)  
- timestamp  
- domain ownership  
- access level  

Metadata is used to enforce retrieval rules.

---

## Storage Rules

- Raw data remains in its domain-specific storage  
- Embeddings are stored in a shared index  
- No duplication of raw data in embedding storage  
- Embeddings may reference but not contain full source data  

---

## Retrieval Behavior

### Domain Apps (Training, Mind, Learn)

- May:
  - perform semantic search across all embeddings  
  - identify relevant cross-domain context  
  - retrieve summaries or scoped data  

- Must NOT:
  - retrieve unrestricted raw data from other domains  
  - bypass domain-specific access rules  

---

### Connect

- May:
  - retrieve embeddings across all domains  
  - assemble multi-domain context  
  - perform deep cross-domain reasoning  

- Acts as:
  - global retrieval layer  
  - orchestration surface  

---

## Context Reconstruction

Embeddings are used to:

- locate relevant information  
- reconstruct context dynamically  
- avoid transferring full chat history  

---

## Rerouting Integration

When rerouting:

- embeddings from the source conversation are reused  
- the target app retrieves relevant context  
- continuity is preserved without raw transfer  

---

## Anti-Corruption Rules

The system must NOT:

- treat embeddings as source truth  
- bypass domain ownership rules  
- store sensitive raw data in embedding form  
- over-retrieve irrelevant context  

---

## Summary

The Shared Embedding System:

- indexes all domain content semantically  
- enables cross-app discovery  
- preserves domain ownership  
- supports seamless continuity  

It ensures:

- efficient memory usage  
- scalable context retrieval  
- safe cross-domain interaction  

Embeddings are the map, not the territory.

## Related
- [[EVO Architecture Bible]]
- [[EVO Wiki — One Alice, Many Rooms.md]]
- [[EVO — Adapter Training System.md]]
- [[EVO — Cognition Layer.md]]
- [[EVO — Context Layer.md]]
- [[EVO — Cross-App Context Continuity.md]]
- [[EVO — Global Adapter Distribution Model.md]]
- [[EVO — Pane Pack Architecture.md]]
^[wiki-native — no upstream source]
