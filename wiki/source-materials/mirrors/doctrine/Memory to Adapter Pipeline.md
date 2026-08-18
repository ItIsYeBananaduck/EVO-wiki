---
title: Memory to Adapter Pipeline
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Memory to Adapter Pipeline.md"]
updated: 2026-07-24
---

# Memory to Adapter Pipeline
## Purpose

Define how Alice's memory layer feeds adapter updates without skipping the journal stage.

## Pipeline

Logs → Journal Entries → Linked Patterns → Training Data → Adapter Updates

## Rule

Memory first. Compression later.

Alice should not train adapters directly from raw daily experience without first passing through reflection and journaling.

## User Adapter Cadence

- updated weekly

Reason:

User-specific adaptation benefits from a shorter loop because the goal is to improve personalization while still allowing time for journal formation and correction.

## Global Adapter Cadence

- updated biweekly or monthly

Reason:

Global behavior should move more slowly and only after enough repeated cross-user evidence has accumulated.

## Journal Role

Journals are the learning layer.

Adapters are the compression layer.

## Correction Role

When Alice is corrected:

- the user journal is revised
- the work journal records what reasoning went wrong
- that reasoning lesson may later shape training data for model improvement

## Principle

Adapters should distill proven patterns, not replace the visible memory system that makes learning inspectable and correctable.

---

Related notes: [[EVOLoRA Mesh — Adapter Creation Pipeline]]

## Related

^[{src_rel}]
