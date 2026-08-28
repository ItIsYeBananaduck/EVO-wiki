---
title: "EVOS1-84 — Diff Scope from Last Merge to Current Branch"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-08-19
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-84 — Diff Scope from Last Merge to Current Branch

## Scope definition

- **Current branch tip:** `556b5976eb55a6a80b8e4359989012955abe9b9f`
- **Last merge commit before tip:** `9cd1b059eb89df83c04db5266781b5c36fff157e`
- **Diff range used for scope:** `9cd1b059eb89df83c04db5266781b5c36fff157e..556b5976eb55a6a80b8e4359989012955abe9b9f`

## Changed files (complete)

1. `docs/audits/EVOS1-96-on-device-vs-server-responsibilities-adapter-updates.md`

## Grouped touched areas

### Packages (`packages/*`)

- No package files changed in this range.

### Apps (`apps/*`, `app/*`, platform app entrypoints)

- No app files changed in this range.

### Surface areas

- **Audit / governance documentation**
  - `docs/audits/EVOS1-96-on-device-vs-server-responsibilities-adapter-updates.md`

## Downstream audit-ready summary

This diff scope contains **documentation-only changes** under `docs/audits` and does **not** include source changes in package or app runtime surfaces. Downstream review can focus on governance/audit content alignment without requiring package build/type validation for this specific range.

## Repro commands

```bash
# Find the merge base between testing branch and current HEAD
git merge-base testing HEAD
git diff --name-only 9cd1b059eb89df83c04db5266781b5c36fff157e..556b5976eb55a6a80b8e4359989012955abe9b9f
```