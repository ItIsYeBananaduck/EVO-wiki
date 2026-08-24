---
title: Living Note Creation Flow
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Living Note Creation Flow.md"]
updated: 2026-07-24
---

# Living Note Creation Flow
## Purpose

Define the governed flow for turning chat, user intent, observations, or explicit slash commands into approved Living Notes inside EVOconnect.

This flow exists so Alice can help the user think out loud, refine ideas, propose structure, propose links, and only then create trusted knowledge.

## Core Principle

**Living Notes are not created directly from chat. They are drafted first, then approved.**

The user speaks naturally. Alice identifies durable knowledge. Alice drafts the note. Alice proposes links. The user approves or edits. Only then is the Living Note created.

## Scope

This flow applies to Connect-owned Living Notes.

It does not apply to:

- automatic journal entries
- training logs
- mind logs
- Learn temporary memory
- raw external agent traces
- unreviewed observations

Those may inform a draft, but they do not become Living Notes without approval.

## Entry Points

### Passive Suggestion

Alice may detect that a conversation contains durable knowledge.

Examples:

- a system rule
- an architecture decision
- a domain boundary
- a workflow pattern
- a correction to existing doctrine
- a repeated user preference
- an implementation constraint
- a cross-domain coordination rule

Alice must ask before drafting:

> This feels like something worth preserving as a Living Note. Want me to draft it?

If the user says yes, Alice starts the draft stage.

### Slash Command

The user may explicitly trigger note drafting with:

```plain text
/note
```

The slash command bypasses passive detection and starts the draft stage immediately.

The command may include inline context:

```plain text
/note Living Notes should require approval before creation.
```

Or it may refer to recent conversation:

```plain text
/note this
```

### Promotion From Cognition

Alice may propose a Living Note draft from:

- journal insights
- repeated log patterns
- validated observations
- external agent observations
- workflow outcomes

Promotion still requires draft approval.

## Capture Sensitivity

Passive suggestion frequency must be adjustable.

Suggested modes:

### Low

Only suggest notes for major system decisions, durable rules, or explicit user corrections.

### Medium

Suggest notes for important concepts, architecture decisions, recurring patterns, or cross-domain rules.

### High

Power-user capture mode. Suggest notes more often during active design, planning, architecture, or doctrine sessions.

### Manual Only

Never passively suggest. Only create drafts when the user uses `/note` or explicitly asks.

## Flow Overview

```plain text
Signal detected or /note called
→ gather source context
→ classify note candidate
→ draft note
→ propose links
→ show draft to user
→ user edits / approves / rejects
→ create Living Note
→ index note
→ update maps/backlinks
→ record creation trace
```

## Step 1 — Gather Source Context

Alice gathers only the minimum useful context needed to draft the note.

Sources may include:

- current chat message
- recent conversation window
- selected text
- referenced task/project/phase
- related existing notes
- relevant domain map summaries
- journal insight or observation summary

Alice should avoid loading large note bodies unless needed.

## Step 2 — Classify Candidate

Alice classifies the candidate before drafting.

Possible candidate types:

- architecture note
- doctrine note
- workflow note
- decision note
- domain map update
- knowledge map update
- project memory
- task context note
- talent support note
- external observation note

If the candidate belongs to Training, Mind, or Learn domain knowledge, Alice should not create a Connect Living Note unless the content is cross-domain, operational, or workflow-related.

## Step 3 — Generate Draft

Alice creates a draft object, not a final note.

Required draft fields:

- proposed title
- one-line purpose
- short summary
- refined body
- original phrasing or source context when useful
- proposed domain
- proposed type
- proposed tags
- proposed related notes
- proposed related projects
- proposed related phases
- proposed related tasks
- retrieval anchors
- map placement
- governance notes or constraints

The draft must be readable by the user and structured enough for Alice to index later.

## Step 4 — Propose Links

Alice proposes links before approval.

Link targets may include:

- Living Notes
- domain maps
- knowledge maps
- projects
- phases
- tasks
- talents
- workflows
- related doctrine
- external agent observations

Each proposed link should include a short reason.

Example:

```plain text
Proposed link: EVO Cognition Layer
Reason: Living Notes are defined as Connect’s approved knowledge layer inside the broader cognition system.
```

## Step 5 — User Review

The user may:

- approve as-is
- edit the draft
- ask Alice to revise
- remove proposed links
- add links
- reject the draft
- save it as a scratch instead

Until approved, the draft is not trusted knowledge.

## Step 6 — Create Living Note

After approval, Alice creates the Living Note.

Creation commits:

- title
- body
- summary
- source context
- links
- retrieval anchors
- map placement
- metadata

The new note is marked:

- Status: Active
- Drive Sync Status: Not Synced
- Source: Chat, unless another source is more accurate

## Step 7 — Index And Map

After creation, Alice makes the note navigable.

Required actions:

- add retrieval anchors
- connect backlinks
- place note in relevant domain or knowledge map
- ensure summaries exist
- make note eligible for targeted RAG

Navigation should remain map-first and token-light.

## Step 8 — Record Creation Trace

Alice records why the note exists.

Creation trace should include:

- source type
- triggering chat or command
- approval moment
- links accepted or rejected
- whether this came from passive suggestion, `/note`, or cognition promotion

This trace helps Alice explain or revise the note later.

## Failure Cases

### User rejects draft

Do not create the note.

Optional: remember only that the user rejected this capture type if useful for capture sensitivity tuning.

### User says it belongs elsewhere

Route it to the proper domain or memory type.

Example:

- training performance pattern → Training cognition
- emotional pattern → Mind cognition
- temporary lesson context → Learn temporary memory
- cross-domain workflow rule → Connect Living Note

### Existing note already covers it

Alice should propose updating or linking the existing note instead of creating a duplicate.

Updates still require approval and must set Drive Sync Status to Not Synced.

### Draft lacks enough context

Alice asks for the smallest clarification needed, or creates a minimal draft with uncertainty clearly marked.

## Governance Rules

- No silent Living Note creation.
- `/note` triggers drafting, not direct creation.
- Passive detection asks before drafting.
- Approval is required before creation.
- Proposed links must be shown before approval.
- Existing notes should be reused or updated when possible.
- Domain-owned knowledge should stay in its domain unless it becomes operational Connect knowledge.
- Created or updated notes must be marked Not Synced for Drive export.

## Relationship To Create Living Note Talent

This flow is executed by the Create Living Note Talent.

The talent owns:

- passive detection
- slash command handling
- draft generation
- link proposal
- approval handling
- note creation
- indexing and map updates

## Related Notes

- [EVO Cognition Layer](https://www.notion.so/354c72bad01381aeb193f1e9b2bd2b1f)
- [Living Notes — Connect Knowledge System](https://www.notion.so/354c72bad013815bb055d0e1f13816fa)
- [Create Living Note Talent](https://www.notion.so/354c72bad013816da597eeb0afbd3ebd)
- [Token-Light Knowledge Navigation](https://www.notion.so/354c72bad013815cb880ce154948e879)
- Domain Maps & Knowledge Maps
- [Slash Command System (Connect)](https://www.notion.so/354c72bad01381fd887eff3aa8ac0b27)

## Summary

Living Note creation is a governed collaboration loop.

The user talks. Alice refines. Alice proposes links. The user approves. Connect receives durable, structured, navigable knowledge.
