---
title: User Memory MOC
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/User Memory MOC.md"]
updated: 2026-07-24
---

# User Memory MOC
## Purpose

This document serves as the navigation map for user-specific memory in Alice.

User memory captures what Alice learns about a specific person over time while keeping that memory visible, challengeable, and source-backed.

---

## Core Notes

- [[Alice Journal System]]
- [[User Journal System]]
- [[Journal Entry Schema]]
- [[Daily Reflection Engine]]

---

## Core Questions

- What should Alice remember about the user?
- What qualifies as durable memory instead of temporary session context?
- How does the user challenge memory without directly editing it?
- How should source links resolve in the user-facing UI?

---

## Principle

User memory exists to improve continuity and personalization without becoming opaque or manually rewritten by the user.

---

## Summary

This MOC organizes the part of Alice’s memory system that is about understanding the user better over time.

---

## Vector Memory Indexing Policy

Only cognition-relevant artifacts are embedded into vector memory. Raw system exhaust and transport events are excluded.

### Indexing Decision Rules

| Artifact type | Default decision |
|---|---|
| `journal_entry` | `index` |
| `journal_summary` | `index` |
| `observation` | `index` if approved |
| `user_preference` | `index` |
| `plan_decision` | `index` |
| `raw_log` | `skip` |
| `draft_unapproved` | `quarantine` |
| `transport_event` | `skip` |

### Pre-embedding Requirements

- Text must be normalized and summarized before embedding (no raw event payloads)
- Each `VectorCandidate` must include: `candidate_id`, `user_id`, `app_domain`, `artifact_type`, `created_at`, `source_ref`, `text`
- `summary_level` must be declared: `primary | detail | chunk`
- Deduplication required — same source_ref + version must not be re-indexed unless content changed

### Hard Boundaries (do not index)

- Raw training logs
- Raw inference traces
- Transport/IPC envelopes
- Draft journal entries pending user review
- Any artifact with `approved: false`

## Related
