---
title: EVO — Cross-App Context Continuity
type: concept
tags: [context, evo]
sources:
  - source-materials/mirrors/doctrine/EVO — Cross-App Context Continuity.md
updated: 2026-07-23
---
# EVO — Cross-App Context Continuity

> NOTE: This is a canonical doctrine note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

> RULE: All `related` entries must use Obsidian wiki link format.

---

## Purpose

Define how Alice maintains conversational continuity when moving between apps without transferring full chat history.

Continuity must feel seamless while preserving domain boundaries and minimizing token usage.

---

## Core Principle

Continuity is maintained through **semantic memory**, not raw conversation transfer.

Alice does not carry conversations between apps.  
She reconstructs context using shared embeddings and structured retrieval.

---

## Definitions

**Embedding**  
A semantic representation of a message, note, or concept used for retrieval.

**Context Continuity Packet**  
A lightweight bundle of information used to rehydrate conversation state in a new app.

**Rerouting**  
The process of moving a conversation from one domain to another.

---

## System Structure

User Message  
→ Embedding Created  
→ Stored in Shared Embedding Space  
→ Reroute Triggered  
→ Retrieval in Target App  
→ Context Reconstructed  
→ Conversation Continues  

---

## Context Continuity Packet

When rerouting occurs, Alice must construct a packet containing:

- current user intent  
- last meaningful user message  
- topic embedding  
- source app  
- relevant retrieved context (notes, summaries, journals)

The packet must be:

- minimal  
- domain-safe  
- semantically complete  

---

## Rules

- Full chat history must NOT be transferred between apps  
- Context must be reconstructed via embeddings  
- Retrieval must respect domain access rules  
- Only relevant context may be included  

---

## Retrieval Behavior

### Domain Apps (Training, Mind, Learn)

- Retrieve:
  - relevant summaries
  - scoped cross-domain context
- Must NOT:
  - pull full cross-domain histories
  - bypass domain rules

---

### Connect

- May retrieve:
  - full cross-domain context
  - multiple domain perspectives
- Acts as:
  - cross-domain assembler
  - orchestration layer

---

## Rerouting Flow

1. User asks out-of-domain question  
2. Alice identifies mismatch  
3. Alice offers reroute:
   - target domain app
   - or Connect (if available)  
4. On approval:
   - create context continuity packet  
   - embed + store  
   - open new chat in target  
5. Target app:
   - retrieves context via embeddings  
   - continues conversation  

---

## User Experience Principle

Rerouting must feel like:

> Alice moved to a different room and continued the conversation.

Not:

> A new conversation started.

---

## Anti-Corruption Rules

The system must NOT:

- duplicate large context blocks  
- rely on raw chat transfer  
- leak domain-restricted data  
- break execution boundaries  

---

## Summary

Cross-app continuity is achieved through:

- shared embeddings  
- minimal context packets  
- governed retrieval  

This enables:

- seamless user experience  
- efficient context transfer  
- preservation of domain boundaries  

Continuity is reconstructed, not carried.

## Related
- [[EVO Architecture Bible]]
- [[EVO Wiki — One Alice, Many Rooms.md]]
- [[EVO — Adapter Training System.md]]
- [[EVO — Cognition Layer.md]]
- [[EVO — Context Layer.md]]
- [[EVO — Global Adapter Distribution Model.md]]
- [[EVO — Pane Pack Architecture.md]]
- [[EVO — Shared Embedding System.md]]
^[source-materials/mirrors/doctrine/EVO — Cross-App Context Continuity.md]
