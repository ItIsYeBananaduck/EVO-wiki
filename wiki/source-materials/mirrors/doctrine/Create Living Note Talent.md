---
title: Create Living Note Talent
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Create Living Note Talent.md"]
updated: 2026-07-24
---

# Create Living Note Talent
## Purpose

Define the Connect talent responsible for turning meaningful chat, explicit `/note` commands, and approved cognition promotions into governed Living Note drafts and approved Living Notes.

This talent implements the Living Note Creation Flow.

## Core Principle

**The Create Living Note Talent creates drafts first, not notes.**

A Living Note is only created after the user approves the draft and its proposed links.

## Talent Scope

This talent belongs to EVOconnect.

It may reference knowledge from EVOtraining, EVOmind, EVOlearn, and Alice Core, but it must not absorb domain-owned knowledge unless the content becomes Connect-operational knowledge.

## Entry Points

### Passive Detection

The talent may run after meaningful conversation turns when Alice detects durable knowledge.

The talent must ask before drafting.

### Slash Command

The talent runs immediately when the user invokes:

```plain text
/note
```

The command may include inline content or refer to recent chat context.

### Cognition Promotion

The talent may be invoked when a journal insight, validated observation, external model observation, or repeated workflow pattern should be promoted into Connect’s trusted knowledge layer.

Promotion still requires user approval.

## Inputs

Required or optional inputs may include:

- trigger source: passive, slash command, or cognition promotion
- current user message
- recent chat context
- selected source text
- source app/domain
- referenced project
- referenced phase
- referenced task
- related note candidates
- relevant map summaries
- existing note matches
- capture sensitivity setting

## Outputs

The talent can produce:

- no-op decision
- capture suggestion
- Living Note draft
- revised draft
- approved Living Note
- update proposal for an existing note
- scratch suggestion when the content is useful but not trusted enough
- routing recommendation to another cognition type

## Talent Lifecycle

```plain text
Detect or receive command
→ gather minimal context
→ classify candidate
→ duplicate check
→ propose capture or draft
→ generate structured draft
→ propose links
→ request approval
→ create or update note
→ index and map
→ record trace
```

## Step 1 — Detect Or Receive Trigger

### Passive Mode

Alice evaluates whether the conversation contains durable knowledge.

Signals include:

- system rule
- architecture decision
- correction to doctrine
- cross-domain boundary
- workflow pattern
- talent behavior
- repeated user preference
- external model observation worth preserving

If the signal passes the user’s capture sensitivity threshold, Alice asks whether to draft.

### Slash Mode

If the user invokes `/note`, Alice skips passive detection and begins drafting.

### Promotion Mode

If another cognition layer proposes promotion, Alice treats the source as a candidate, not trusted knowledge.

## Step 2 — Gather Minimal Context

Alice gathers the smallest useful context bundle.

Priority order:

1. current message or slash command payload
2. recent nearby chat turns
3. referenced project/task/phase metadata
4. summaries of likely related notes
5. domain or knowledge map summaries
6. full note sections only if needed

The talent must avoid loading entire domains or large note bodies by default.

## Step 3 — Classify Candidate

Alice classifies what kind of draft this should become.

Candidate classes:

- architecture note
- doctrine note
- decision note
- flow note
- talent support note
- project memory
- task context note
- domain map update
- knowledge map update
- external observation note
- routing-only memory

If the content belongs to another app’s domain knowledge, Alice routes it rather than creating a Connect Living Note.

## Step 4 — Check For Existing Notes

Before drafting a new Living Note, Alice checks whether an existing note already covers the concept.

Possible outcomes:

- create new note
- update existing note
- link existing note only
- mark conflict for review
- route to another domain cognition layer

Duplicate prevention is required.

## Step 5 — Generate Draft

Alice generates a structured draft.

Required draft fields:

- proposed title
- one-line purpose
- short summary
- refined content
- original user phrasing when useful
- source context
- proposed domain
- proposed type
- proposed tags
- retrieval anchors
- proposed related notes
- proposed related projects
- proposed related phases
- proposed related tasks
- proposed map placement
- governance notes
- uncertainty notes, if any

The draft is not trusted until approved.

## Step 6 — Propose Links

Alice proposes links with reasons.

Each proposed link should include:

- target name
- target type
- relationship reason
- confidence level if useful

Example:

```plain text
Link: EVO Cognition Layer
Reason: Defines where Living Notes sit inside Alice’s broader cognition system.
```

## Step 7 — Present Draft For User Review

Alice presents the draft in a compact, readable format.

The user can:

- approve
- reject
- revise
- remove links
- add links
- convert to scratch
- route elsewhere

## Step 8 — Approval Gate

The talent may only create or update a Living Note after explicit user approval.

Approval commits:

- content
- links
- metadata
- map placement
- indexing eligibility

## Step 9 — Create Or Update Note

After approval, Alice creates or updates the relevant EVOnotes page.

Required properties:

- Status: Active
- Drive Sync Status: Not Synced
- Source: Chat, unless another source is more accurate
- Domain: EVOconnect, unless the approved note is intentionally cross-app or core architecture
- Type: matching approved classification

If updating an existing note, Drive Sync Status must be changed to Not Synced.

## Step 10 — Index And Map

After creation or update, Alice prepares the note for fast navigation.

Required actions:

- ensure one-line purpose exists
- ensure summary exists
- add retrieval anchors
- apply or update related note links
- update domain or knowledge map if required
- make the note eligible for targeted RAG

## Step 11 — Record Trace

Alice records why the note exists.

Trace should include:

- trigger source
- source context
- approval moment
- accepted links
- rejected links, if useful
- update vs new note decision
- routing decisions

## Capture Sensitivity Behavior

The talent must respect the user’s selected capture sensitivity.

### Manual Only

Only runs from `/note` or explicit user instruction.

### Low

Suggests only for durable rules, major decisions, and explicit corrections.

### Medium

Suggests for important concepts, architecture decisions, and recurring patterns.

### High

Suggests frequently during active design, doctrine, architecture, or planning sessions.

## Failure Handling

### User rejects draft

Do not create a Living Note.

Optionally tune future capture suggestions.

### Existing note conflict

Propose update or conflict review instead of creating a duplicate.

### Wrong domain

Route to the correct cognition layer.

### Missing context

Ask for the smallest useful clarification or produce a draft with uncertainty clearly marked.

### Link uncertainty

Show uncertain links separately from high-confidence links.

## Governance Rules

- The talent may draft without committing knowledge.
- The talent may not silently create Living Notes.
- The talent may not silently update Living Notes.
- `/note` triggers drafting, not direct creation.
- Proposed links must be visible before approval.
- Domain-owned knowledge should stay in its domain unless approved as Connect-operational knowledge.
- Updates must set Drive Sync Status to Not Synced.

## Relationship To Other Systems

### EVO Cognition Layer

Defines the broader memory and knowledge system this talent operates inside.

### Living Notes — Connect Knowledge System

Defines what Living Notes are and why they are Connect-owned.

### Living Note Creation Flow

Defines the lifecycle this talent executes.

### Token-Light Knowledge Navigation

Defines how the talent should find related notes without burning context.

### Slash Command System

Defines `/note` and future commands that trigger talents.

## Summary

The Create Living Note Talent is the governed capture mechanism for Connect.

It lets the user think naturally while Alice turns high-value conversation into structured, linked, approved, navigable knowledge.

## Related
