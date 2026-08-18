---
title: Living Notes vs Scratches
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Living Notes vs Scratches.md"]
updated: 2026-07-24
---

# Living Notes vs Scratches
## Core Idea

Not all notes are equal.

EVOconnect distinguishes between three note classes:

- Scratches → temporary user scratchpad notes
- Comments → static project-bound notes for the user
- Live Notes → maintained project-bound notes for Alice and the system

These note classes differ by:

- ownership
- sync behavior
- update responsibility
- project attachment
- system relevance

## Scratch

A Scratch is a temporary user note.

It is:

- short-term
- static
- local-first
- not auto-updatable
- not tied to projects, tasks, phases, or subtasks

### Role

- scratchpad
- rough capture
- temporary workspace
- random note for the user

### Principle

A Scratch is explicitly for the user, not for Alice’s maintained working memory.

### Pins and Sync

Pins are for UI visibility, not note type.

A pinned Scratch remains a Scratch.

Scratch sync behavior is controlled by user policy:

- unpinned scratches are local-only by default
- pinned scratches are syncable by default
- the user may override this with a global policy:
    - sync no scratches
    - sync pinned scratches only
    - sync all scratches

## Comment

A Comment is a structured static note tied to project structure.

It is:

- static
- project-bound
- user-oriented
- tied to a project, phase, task, or subtask
- visible upward through the hierarchy
- not auto-updatable

### Role

- structured user context
- rationale
- reminders
- observations tied to execution structure

### Principle

A Comment belongs to the project system, but it is still primarily for the user rather than for Alice’s maintained note layer.

## Live Note

A Live Note is a maintained, system-relevant note.

It is:

- syncable
- auto-updatable
- tied to a project, phase, task, or subtask
- potentially temporary, long-term, permanent, or archived
- source material for Alice

### Role

- maintained project memory
- evolving reference
- structured source material
- note layer Alice is responsible for keeping current

### Principle

A Live Note is not just saved for the user.

It is part of Alice’s maintained working knowledge system.

## Key Distinction

Scratches are for the user’s temporary mind.

Comments are for the user’s structured project context.

Live Notes are for Alice’s maintained project memory.

## Sync Rules

Scratch:

- not synced to cloud
- local only

Comment:

- attached to project structure
- stored with project context
- static

Live Note:

- synced
- maintained
- part of active project memory

## Update Rules

Scratch:

- no automatic updates
- only changes if the user edits it directly

Comment:

- static by default
- may be edited manually
- not maintained automatically

Live Note:

- maintained by Alice
- may be updated through conversation, task flow, or system-backed refresh logic

## Attachment Rules

Scratches are not attached to project structure.

Comments and Live Notes are attached to exactly one canonical level:

- Project
- Phase
- Task
- Subtask

They remain visible at ancestor levels above that attachment point.

## Visibility Model

Scratch:

→ local only

Comment:

→ visible at home level and upward through ancestors

Live Note:

→ visible at home level and upward through ancestors

No duplicate copies are created.

There is one canonical note record.

## Principle

Not everything should become maintained knowledge.

But everything should have a place to begin.

## Related

^[{src_rel}]
