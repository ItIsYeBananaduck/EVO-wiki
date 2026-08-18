---
title: Token-Light Knowledge Navigation
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Token-Light Knowledge Navigation.md"]
updated: 2026-07-24
---

# Token-Light Knowledge Navigation
## Purpose

Define how Alice navigates large knowledge pools quickly, accurately, and with minimal token use.

This doctrine exists because EVO is designed to give Alice access to a massive pool of knowledge without forcing her to load everything into context.

## Core Principle

**Alice should navigate before she searches.**

She should not blindly run semantic search across the whole knowledge base unless no better map or link path exists.

## Retrieval Doctrine

Alice retrieves knowledge in this order:

```plain text
Intent → Domain → Map → Backlinks → Shortlist → Targeted RAG → Exact Context → Response or Action
```

Backlinks get Alice to the likely neighborhood.

RAG gets Alice to the exact house.

Full notes are cold storage, not the default working context.

## Why This Matters

Large knowledge systems fail when retrieval becomes blind.

Blind retrieval causes:

- token waste
- noisy context
- wrong domain mixing
- hallucinated connections
- slow responses
- poor mobile performance

Map-first retrieval keeps Alice fast, grounded, and cheap to run.

## Navigation Layers

### Level 0 — App / Domain Awareness

Alice first determines which domain owns the request.

Examples:

- Training request → EVOtraining cognition
- Mind request → EVOmind cognition
- Learn request → EVOlearn cognition
- Workflow / task / agent request → EVOconnect cognition
- Cross-domain request → Connect global domain map

Training, Mind, and Learn are easier because they are usually bounded to their app domain.

Connect is harder because it can coordinate anything. Connect therefore needs a global domain map.

### Level 1 — Domain Map

A domain map tells Alice where categories of knowledge live.

It should include:

- major concepts
- active knowledge maps
- canonical notes
- important workflows
- domain boundaries
- related domains

### Level 2 — Knowledge Map

A knowledge map organizes a specific topic or system area.

Examples:

- Living Notes knowledge map
- Delegator knowledge map
- Talents knowledge map
- Hive knowledge map
- Training cognition map

Knowledge maps help Alice choose a small set of likely notes before using RAG.

### Level 3 — Backlinks And Typed Relationships

Backlinks and typed relationships guide Alice through the graph.

Relationship examples:

- governs
- depends on
- supersedes
- supports
- conflicts with
- implements
- references
- routes to
- promotes to
- belongs to

Typed links are better than raw links because they explain why two things are related.

### Level 4 — Shortlist

Alice creates a shortlist of likely relevant notes or records.

The shortlist should usually be small.

Default target:

- 3 to 7 candidate nodes

Only after shortlisting should Alice run targeted RAG inside those candidates.

### Level 5 — Targeted RAG

RAG should search inside the shortlisted notes, logs, journals, or records.

The goal is to retrieve exact sections, not entire documents.

RAG should answer:

- which paragraph matters?
- which claim matters?
- which correction matters?
- which log event matters?
- which prior decision matters?

### Level 6 — Exact Context Load

Alice loads only the exact context needed.

Default context preference:

1. title
2. one-line purpose
3. short summary
4. relevant section/chunk
5. full note only when needed

## Domain Routing Rules

### EVOtraining

Training owns:

- workout logs
- readiness and strain context
- pose estimation outputs
- OCR workout imports
- training journal entries
- performance trends

Connect may reference Training cognition but should not duplicate it unless the knowledge becomes workflow or coordination knowledge.

### EVOmind

Mind owns:

- emotional logs
- stress logs
- conversational journal entries
- mental pattern observations

Connect may reference Mind cognition but should not absorb it.

### EVOlearn

Learn owns:

- active learning context
- temporary topic memory
- lesson/session context
- learning pattern observations

Temporary learning memory expires when the user no longer wants to learn that subject.

### EVOconnect

Connect owns:

- Living Notes
- workflows
- talent training
- external model observations
- cross-domain maps
- task/project/phase context
- operational knowledge

Connect can navigate across domains, but it does not own all domain knowledge.

## Map Requirements

Each map should be lightweight.

A good map contains:

- name
- purpose
- scope
- owned knowledge areas
- canonical notes
- related maps
- common retrieval anchors
- boundaries

A map should not be a giant document.

It should be an index, not a textbook.

## Note Requirements For Fast Retrieval

Every important note should support progressive loading.

Recommended fields:

- title
- one-line purpose
- short summary
- medium summary when needed
- retrieval anchors
- related notes
- domain
- type
- source
- status
- canonical flag when relevant

Living Notes should also include:

- proposed or approved links
- map placement
- source context
- creation trace
- governance constraints when relevant

## Retrieval Anchors

Retrieval anchors are phrases Alice is likely to search for later.

They should include:

- user phrasing
- normalized system phrasing
- synonyms
- domain-specific terms
- command words
- workflow names

Example:

```plain text
Retrieval Anchors:
- chat to note
- /note
- Living Note draft
- note approval
- proposed links
- governed capture
```

## Progressive Loading Rule

Alice should load knowledge progressively:

```plain text
Map → Note card → Summary → Relevant chunk → Full note only if required
```

This keeps retrieval fast and token-light.

## Search Budget Rules

Default behavior:

- choose domain first
- use maps before search
- prefer summaries over full notes
- shortlist before RAG
- load 3 to 7 likely nodes
- retrieve exact chunks
- avoid full-domain scans

Escalation behavior:

- if the first domain is wrong, check adjacent domains
- if multiple domains are plausible, check 2 to 3 domain maps briefly
- if no map exists, use semantic search as fallback
- if search is noisy, ask for clarification or narrow by project/task/domain

## Connect Global Domain Map

Connect needs a global map because it can coordinate across all domains.

The global map should include:

- Core Platform
- EVOconnect
- EVOtraining
- EVOmind
- EVOlearn
- Governance
- Cross-App systems
- Alice Core
- Delegator
- Talents
- Hive
- External Agents
- User Projects

Connect uses this map to decide where to route questions and actions.

## RAG Boundary Rules

RAG should not be the first move unless:

- there is no known map
- the user asks for broad search
- the topic is unknown
- the system cannot classify the domain
- the user explicitly asks to search everything

Otherwise, RAG should be scoped by map and backlinks.

## Failure Handling

### Wrong domain selected

Alice should acknowledge the mismatch and route to the better domain.

### Too many candidate notes

Alice should narrow by domain, project, task, note type, or time.

### No relevant map exists

Alice should use semantic search as fallback and suggest creating or updating a map afterward.

### Conflicting notes found

Alice should surface the conflict and prefer canonical, active, recently reviewed notes.

### Outdated notes found

Alice should prefer active notes and treat superseded notes as historical context only.

## Governance Rules

- Alice must not treat every retrieved item as equally trusted.
- Canonical and active notes outrank drafts and superseded notes.
- Living Notes require approval before becoming trusted context.
- Journals are adaptive memory, not absolute truth.
- Logs are factual events, not interpreted knowledge.
- Observations require interpretation or validation.
- Domain-owned knowledge should stay in its domain.

## Relationship To Living Notes

Living Notes are optimized for this navigation model.

They should be created with:

- summaries
- retrieval anchors
- proposed links
- map placement
- source context

This makes them fast for Alice to find and cheap to use.

## Relationship To Slash Commands

Slash commands can bypass passive detection, but they do not bypass navigation.

Example:

```plain text
/note this
```

Alice still checks existing notes, maps, and likely links before drafting.

## Relationship To Talents

Talents should use the same navigation doctrine.

A talent should not load the whole knowledge base.

It should retrieve only the minimum context needed to execute safely and correctly.

## Summary

Token-light navigation is the retrieval backbone of EVO cognition.

Alice uses maps and backlinks to get close, then targeted RAG to retrieve exact knowledge.

This lets EVO scale to a massive knowledge pool without becoming slow, expensive, or noisy.

## Related

^[{src_rel}]
