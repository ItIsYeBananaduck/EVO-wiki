---
title: User Journal System
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/User Journal System.md"]
updated: 2026-07-24
---

# User Journal System
## Purpose

Define the user-specific memory layer Alice maintains about an individual user.

## Scope

The User Journal stores durable observations about the user, including:

- preferences
- recurring goals
- stable patterns
- communication style
- friction points
- tone cues
- behavior signals

## Allowed Sources

- EVOtraining logs
- EVOmind logs and journal entries
- EVOlearn interaction history
- selective Connect interactions

## Write Rules

Alice should only create a user journal entry when:

- a pattern repeats
- an observation appears stable
- the information is useful across sessions
- a correction materially changes understanding of the user

Alice should not write:

- one-off moments
- temporary mood states
- low-signal observations
- session-only context with no durable value

## User Controls

The user may:

- view entries
- delete entries
- challenge entries

The user may not directly edit entries.

## Correction Handling

When an entry is challenged:

- the user journal is revised through a new or replacement entry
- the old understanding is not manually rewritten by the user
- the reasoning failure is logged separately in the Work Journal

## Principle

The User Journal exists to help Alice understand this user better over time without turning user memory into an opaque or user-edited blob.

## Related
