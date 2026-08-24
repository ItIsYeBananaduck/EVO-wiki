---
title: Note Lifecycle — Connect
type: concept
tags: [lifecycle, note-lifecycle]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# Note Lifecycle — Connect

## Purpose

Define how Scratches, Comments, and Live Notes are created, refined, maintained, and promoted within Connect.

---

## Core Principle

Not all notes follow the same lifecycle.

Scratches, Comments, and Live Notes differ in:

- attachment
- update responsibility
- sync behavior
- system importance

---

## Note Classes

- Scratch  
- Comment  
- Live Note  

---

## Scratch Lifecycle

Conversation or manual typing → Scratch → optional save → optional revisit → optional promotion

### Characteristics

- local-first  
- static  
- short-term  
- not tied to project structure  
- not auto-updated  
- not synced  

### Role

Scratch is the user’s temporary workspace.

### Promotion

Scratch may be promoted to:

- Comment  
- Live Note  

---

## Comment Lifecycle

Conversation or explicit attachment → Draft Comment → Approval → Active Comment

### Characteristics

- project-bound  
- static  
- structured  
- user-oriented  
- visible upward  
- not auto-maintained  

### Role

Structured context tied to execution.

---

## Live Note Lifecycle

Conversation or explicit attachment → Draft → Approval → Active → Maintenance → Evolution

### Characteristics

- project-bound  
- synced  
- maintained by Alice  
- source material  

### Role

Alice’s maintained project memory.

---

## Explicit Creation

- `@target` → Live Note  
- `@#target` → Comment  
- `@/target/target` → multi Live Note  
- `@#/target/target` → multi Comment  

No operator → Scratch

---

## Attachment

Notes attach to one level:

- Project  
- Phase  
- Task  
- Subtask  

Visible upward only.  

No duplication.

---

## Promotion

- Scratch → Comment / Live Note  
- Comment → Live Note  
- Live Note → higher level (if scope expands)  

---

## Principle

Scratch = place to think  

Comment = place to structure  

Live Note = place to maintain

## Related
- [[Note Lifecycle — Connect]]
- [[Scratch → Living Note Promotion System]]
- [[Living Notes — Connect Knowledge System]]
^[wiki-native — no upstream source]
