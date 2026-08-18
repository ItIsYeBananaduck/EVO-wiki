---
title: Connect Notes System
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Connect Notes System.md"]
updated: 2026-07-24
---

# Connect Notes System
## Purpose

Define how notes work inside Connect and how they relate to Alice’s journal system.

## Core Principle

Notes in Connect begin from conversation.

Alice does not silently create official notes.

She suggests note creation, drafts the note in chat, and the note only becomes official after user approval.

## Note Creation Flow

1. Conversation produces a meaningful idea.
2. Alice asks whether the user wants a note.
3. If yes, Alice drafts the note in chat.
4. The user reviews and refines it conversationally.
5. Once approved, the note becomes official.
6. The note keeps a backlink to the originating chat.

## What a Note Is

A note is not a raw conversation slice.

It is a cleaned, restructured knowledge object derived from conversation and linked back to its source.

## Note Classes

Connect supports three note classes:

- Scratch
- Comment
- Live Note

### Scratch

A Scratch is a temporary, local-first user note.

It is not tied to project structure, is not synced, and is not maintained by Alice.

### Comment

A Comment is a static note tied to a project, phase, task, or subtask.

It is structured context for the user, but it is not automatically maintained.

### Living Note

A Living Note is a maintained note tied to project structure.

It is synced, updateable, and treated as source material for Alice.

## Note Responsibility Model

Scratches are for the user’s temporary thinking.

Comments are for the user’s structured project context.

Live Notes are part of Alice’s maintained project memory.

## Smart vs Simple Notes

Internally, Alice should distinguish between:

- structured, high-value notes with project relevance
- simple remember-this notes with no strong workspace context

If a note request has no clear Domain, Project, Phase, or Task/Subtask relevance, it becomes a lower-value rogue note rather than a deeply integrated structured note.

## Context Assignment

Alice may create notes from any chat context.

The user does not need to be inside a specific Domain, Project, Phase, or Task for the note to be assigned there.

Alice should infer:

- whether the conversation touches a Domain
- whether it belongs to a Project
- whether it is specific to a Phase
- whether it should live at the Task/Subtask level

## Explicit User-Controlled Note Creation

Users must be able to create and attach notes without relying on Alice’s inference.

Connect supports explicit note operators:

- `@target` → single-target Live Note
- `@#target` → single-target Comment
- `@/target/target` → multi-target Live Note
- `@#/target/target` → multi-target Comment

These are not social mentions.

They are note attachment operators.

If the user explicitly uses one of these operators, the system should obey the requested note type and attachment intent rather than reinterpret it.

Alice may validate and resolve the target, but should not silently change the requested note class.

## Canonical Home Rule

Each note has exactly one canonical home.

That home is the most specific level where the note belongs.

Examples:

- a task-specific note lives on that task
- a phase-level note lives on that phase
- a project-level note lives on that project

## Upward Visibility Rule

A note that lives at a specific level remains visible from higher levels.

Example:

A task-level note should still be visible from:

- the Phase note index
- the Project note index
- the Domain note index
- the global note dashboard

The note does not duplicate upward. It is inherited into those views.

## Upward Visibility Inheritance

Comments and Live Notes attach to one canonical level only.

That note remains visible from all ancestor levels above it.

Examples:

- a Project-level note is visible only at the Project level
- a Phase-level note is visible at the Phase and Project levels
- a Task-level note is visible at the Task, Phase, and Project levels
- a Subtask-level note is visible at the Subtask, Task, Phase, and Project levels

This is visibility inheritance, not duplication.

There is always one canonical note record.

## Existing Note Decision Loop

When Alice detects a note-worthy idea, she should check:

7. Do we already have a note on this subject?
8. If yes, does the new information:
- add to it
- refine it
- conflict with it
- branch from it
9. If no, do we need a new note?

## Evolution Rules

When new information relates to an existing note, Alice may:

- extend the note if it is additive
- revise the note if it clearly updates it
- branch into a linked note if it conflicts or diverges
- promote the note upward if its relevance expands beyond its original scope

## Promotion Rule

Notes should begin at the lowest reasonable level.

If the same idea becomes relevant across multiple sibling tasks or broader contexts, Alice may:

- keep one task as a parent and allow inherited visibility
- or promote the note to a Phase, Project, or Domain note and tag it to relevant lower-level items

Promotion should be driven by a combination of:

- scope of applicability
- abstraction level
- reuse potential
- user intent overrides

## Optional Type Layer

Some users may define note types such as:

- Concept
- Spec
- Implementation

These types are optional and user-defined.

Alice should use them when available, but should not require them for all users.

Internally, Alice may still infer note intent even when the user has not defined explicit types.

## Suggestion Style

Alice should adapt how she proposes note creation.

For normal users, she may say:

- Want me to make a note of that?

For power users, she may be more explicit about what kind of note or pattern she detects.

## Task System Integration

Each Project, Phase, Task, and Subtask may expose:

- a Comments section
- a Notes section

Behavior:

- anything created in a Comments section becomes a Comment automatically
- anything created in a Notes section becomes a Live Note automatically

This gives users a deterministic way to create structured notes directly inside execution context without needing to rely on Alice’s conversational suggestion flow.

## Journal Relationship

Notes and journal entries are related but not the same.

Journal:

- Alice’s structured memory
- source-backed reflection
- training-oriented memory layer

Notes:

- user-approved knowledge objects
- intentionally captured and revisitable
- part of the Connect knowledge graph

## Journal to Note Rule

The journal may influence note suggestions.

Alice may detect patterns in the journal that deserve notes.

But the journal may not create official notes without going through the same chat suggestion and approval loop.

## Notes in Journal Entries

By default, journal entries should reference notes rather than duplicating them.

Small contextual summaries may be included when useful, but the journal should remain a clean memory and training surface rather than becoming a second note system.

## Source and Linking

Each official note should maintain:

- backlink to source chat
- links to related notes
- links to relevant task / phase / project / domain anchors
- links to journal entries when relevant

## Principle

Notes in Connect are user-approved, context-aware knowledge objects derived from chat.

They live at a single canonical level, remain visible upward, and connect conversation, execution, and Alice’s memory into one evolving knowledge system.

## Related

^[{src_rel}]
