---
title: Living Notes — Connect Knowledge System
type: concept
tags: [knowledge, living-notes, system]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# Living Notes — Connect Knowledge System

## Purpose

Define Living Notes as the governed knowledge system inside EVOconnect.

Living Notes are not a general wiki and they are not Alice’s whole memory.

They are Connect-owned, approved, structured knowledge objects that help Alice navigate work, coordinate domains, run talents, preserve decisions, and safely reuse context.

## Core Principle

**Living Notes are proposed, refined, linked, and approved. They are never silently created.**

The user talks. Alice refines. Alice proposes links. The user approves. Then the note becomes part of the trusted Connect knowledge graph.

## Relationship To EVO Cognition Layer

Living Notes are one part of the larger EVO Cognition Layer.

The Cognition Layer includes:

- journals
- logs
- observations
- temporary Learn memory
- OCR and pose-derived signals
- external inputs
- external model observations
- Living Notes

Living Notes are Connect-only.

Training, Mind, and Learn own their own domain knowledge. Connect can reference those domains, but should not absorb or duplicate their knowledge unless the knowledge becomes a workflow, operating rule, system map, or cross-domain coordination note.

## What Living Notes Are

Living Notes are high-trust Connect knowledge objects used for:

- system architecture
- workflows
- operating rules
- domain maps
- project memory
- task context
- talent support
- external agent observations
- cross-domain coordination
- durable decisions

Living Notes are designed to be both human-readable and AI-navigable.

## What Living Notes Are Not

Living Notes are not:

- raw chat history
- unreviewed memory
- automatic journal entries
- training logs
- mood logs
- temporary Learn memory
- generic file storage
- a blind vector database
- a place for every domain to dump everything

## Creation Model

Living Notes are created through a governed two-stage process:

```plain text
Conversation or /note command → Living Note Draft → User Approval → Living Note Creation
```

## Entry Points

### Passive Suggestion

Alice may detect that something in conversation is worth capturing.

When this happens, Alice asks before drafting:

> This sounds like something we should preserve as a Living Note. Want me to draft it?

The frequency of these prompts should be adjustable by the user.

Suggested sensitivity levels:

- Low — only major decisions and durable rules
- Medium — important ideas, architecture decisions, recurring patterns
- High — power-user capture mode for active design sessions

### Slash Command

The user can explicitly trigger the note creation flow with:

```plain text
/note
```

The `/note` command bypasses the detection step and starts the draft stage immediately.

## Draft Stage

A Living Note draft is not yet trusted knowledge.

Alice creates a structured draft containing:

- proposed title
- short summary
- refined note body
- original chat context or source excerpt when useful
- proposed domain
- proposed type
- proposed links to notes
- proposed links to projects
- proposed links to phases
- proposed links to tasks
- retrieval anchors
- map placement
- suggested governance notes or constraints

The draft must be editable before approval.

## Approval Stage

The user must approve the draft before it becomes a Living Note.

Approval commits:

- note content
- summaries
- links
- map placement
- retrieval anchors
- indexing eligibility

No Living Note is created silently.

## Linking Model

Alice must propose links during the draft stage.

Possible link targets:

- existing Living Notes
- domain maps
- knowledge maps
- projects
- phases
- tasks
- talents
- workflows
- related doctrine
- external agent observation records

Links are not just decorative. They are navigation paths.

## Navigation Model

Living Notes support fast, token-light navigation.

Alice should navigate in this order:

```plain text
Intent → Domain Map → Knowledge Map → Backlinks → Shortlisted Notes → Targeted RAG → Exact Sections
```

Backlinks get Alice to the likely neighborhood.

RAG gets Alice to the exact part of the knowledge.

Alice should not search the full knowledge pool blindly unless no map or link path exists.

## Indexing Model

Each Living Note should include both human and machine navigation surfaces.

Minimum index fields:

- title
- one-line purpose
- short summary
- medium summary when needed
- retrieval anchors
- related notes
- related projects/tasks when available
- domain
- type
- source
- approval status

Chat-created notes should preserve both:

- how the user phrased the idea
- how Alice normalized the idea

This lets Alice recall notes by user language and system language.

## Relationship To Journals

Journals are reflective learning entries.

They answer:

> What did Alice learn about the user today, and how can she better help in this domain?

Journals may suggest future Living Notes, but journals do not automatically become Living Notes.

Promotion requires:

- proposed Living Note draft
- user approval
- link proposal
- creation into Connect’s Living Notes system

## Relationship To Logs And Observations

Logs record what happened.

Observations interpret what happened.

Living Notes preserve approved knowledge or operating context that should guide future behavior.

Example:

- log: a workflow failed twice
- observation: the failure pattern appears tied to missing project context
- journal: Alice learns that the user prefers explicit project binding
- Living Note: approved rule for Connect task creation requiring project/phase/task context when available

## Governance Rules

- Living Notes require user approval.
- Alice may draft but may not silently create.
- Alice must propose links when creating a draft.
- Alice must preserve enough source context to explain why the note exists.
- Alice should avoid duplicating domain-owned knowledge.
- Connect references domain cognition instead of absorbing it.
- Living Notes are trusted enough to influence workflows and talents after approval.
- Outdated Living Notes should be marked Superseded, not silently overwritten without trace.

## Talent Integration

Living Note creation should be implemented as a Connect talent.

The talent should support:

- passive capture suggestion
- `/note` command activation
- draft generation
- link proposal
- user edit/review
- approval
- creation
- indexing
- map updates

The talent should be observable and tunable so Alice can learn the user’s preferred capture frequency.

## Slash Command Integration

The initial command is:

```plain text
/note
```

Future related commands may include:

- `/task`
- `/project`
- `/decision`
- `/workflow`
- `/link`

Slash commands should trigger governed talents, not direct writes.

---
## Conflict Handling

Living Notes must detect and surface contradictions.

A conflict occurs when:
- two notes cannot both be true
- a new note contradicts an existing note

Conflicts are not patterns.

When a conflict is detected:
- Alice must present both notes
- Alice must explain the contradiction
- The user must choose how to resolve it

Resolution options:
- update existing note
- keep both
- move to scratch for refinement

### A conflict is not an update.

Updates extend or refine existing truth.
Conflicts challenge existing truth.

--- 
## Promotion Constraint

Journal insights, logs, and observations may propose Living Notes.

They must never directly create or update Living Notes.

All promotions require:
- draft
- review
- explicit user approval

--- 

## Summary

Living Notes are Connect’s trusted knowledge surface.

They combine the best parts of a wiki, a knowledge graph, and RAG without becoming a blind memory dump.

The user speaks naturally. Alice refines the idea. Alice proposes structure and links. The user approves. Then Connect gains durable, navigable knowledge.

## Related
- [[Note Lifecycle — Connect]]
- [[Scratch → Living Note Promotion System]]
- [[Living Notes — Connect Knowledge System]]
^[wiki-native — no upstream source]
