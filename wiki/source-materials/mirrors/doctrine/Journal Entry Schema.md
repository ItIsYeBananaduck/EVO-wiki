---
title: Journal Entry Schema
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Journal Entry Schema.md"]
updated: 2026-07-24
---

# Journal Entry Schema
## Purpose

Define the atomic structure of Alice System Journal entries.

This schema applies to Alice-owned cognition entries and is distinct from EVOmind conversational journaling and Connect Living Notes.

---

## Core Entry Shape

Every entry should be:

- small
- atomic
- source-backed
- linkable
- pattern-aware

Entries must be readable on their own but also combinable into larger pattern trails over time.

---

## Entry Types (Conceptual)

Each entry represents one of the following states:

- first observation
- repeated signal
- emerging pattern
- established pattern
- changing pattern
- correction

These states may be tracked internally and do not need to be explicitly labeled in the user-facing entry.

---

## User-Facing Fields

Each entry should include:

- Title
- Body (short, readable)
- Source references (human-readable)
- Timestamp
- Related entries

User-facing entries should communicate meaning clearly without requiring the user to inspect raw logs.

---

## Internal Fields

Alice may track internally:

- confidence / uncertainty
- repetition count
- pattern stage
- contradiction signals
- inference notes
- adapter eligibility flags
- app-domain origin

These fields are not exposed to the user.

---

## Evidence and Source Rules

Entries must distinguish between types of sources.

Evidence sources:

- logs (primary evidence)
- user actions (edits, overrides, selections)

Context sources:

- prior journal entries (pattern trail)
- conversational journaling (user-provided context)

Correction sources:

- user corrections
- revised interpretations

User-facing source links should resolve to human-readable surfaces.

Internal links should resolve to raw data or structured records.

---

## Meaning Layer

Each entry should capture:

- what Alice observed
- what Alice believes it means
- what Alice may do differently going forward

The entry should not only describe events, but interpret them.

---

## Linking Rules

Entries may link to:

- prior related entries (pattern progression)
- correction entries
- superseded interpretations
- supporting evidence

Entries should form chains that represent evolving understanding.

---

## Pattern Trail Rules

Entries must not exist in isolation.

Alice should connect entries over time to represent:

- repeated behavior
- emerging patterns
- established patterns
- changing behavior

Alice must not repeat identical observations as new insights.

Instead, repeated evidence should advance the pattern stage.

Changing patterns should not be treated as conflicts, but as adaptations.

---

## Correction Rules

When a user corrects an entry, the system should record:

- original belief
- evidence used
- user clarification
- what was misunderstood
- updated interpretation

Corrections do not delete history.

They update interpretation.

When logs and user input disagree, the system should bias the user’s explanation while preserving logs as evidence.

---

## Adapter Training Relevance

Entries may contribute to adapter preparation.

Internally, Alice should track:

- whether an entry is eligible for training
- whether sufficient pattern repetition exists
- whether the entry is part of a stable pattern
- whether the entry is part of a transitional pattern

Training data should be derived from:

- log deltas (what changed)
- journal interpretation (what it means)

Entries should not be used for training when:

- the signal is one-off
- the pattern is not established
- a correction is pending or unresolved

---

## Voice Rules

Entries are written in Alice’s first-person voice.

Alice refers to herself as “I.”

Alice refers to the user in third person by name or appropriate reference.

The tone should be:

- reflective
- warm
- tentative when uncertain

Entries should feel like Alice thinking about her user, not a system reporting data.

---

## Principle

Journal entries must remain small, auditable, and composable.

Their power comes from how they connect over time, not how large any single entry becomes.

---

## Cross-App Contract (Cognition Layer)

The canonical `journal_entry` field schema across EVOtraining, EVOmind, EVOlearn, and EVOconnect:

Required fields: `entry_id`, `user_id`, `app_domain`, `created_at`, `effective_date`, `summary`, `learned_about_user`, `help_strategy`, `confidence`, `privacy_tier`, `visibility`, `source_references`, `corrections`, `flags`, `version`

`app_domain` enum: `EVOtraining | EVOmind | EVOlearn | EVOconnect`

### Hard Boundary: Journals are NOT Living Notes

`journal_entry` objects MUST NOT be persisted in Living Notes tables. Living Notes artifacts MUST NOT be treated as canonical journal records.

- **Journals** = synthesized, domain-scoped learning artifacts (Cognition Layer)
- **Logs** = raw/near-raw event records (Ingestion Layer)
- **Living Notes** = user-facing long-lived note artifacts with separate lifecycle

### App Mapper Responsibilities

Each domain app owns the transformation of its raw events → `journal_candidate` objects for cognition-layer materialization into canonical `journal_entry` records:

- **EVOtraining mapper**: workout logs, strain records, StrainSync music logs → journal candidates via `[[EVOtraining — StrainSync System]]`
- **EVOmind mapper**: conversation turns, emotional signals → journal candidates
- **EVOlearn mapper**: learning events, quiz results, retention signals → journal candidates
- **EVOconnect mapper**: task outcomes, delegation events → journal candidates

See also: [[EVOmind — Signal Model]], [[Ingestion-Pipeline-Reuse]]

## Related
