---
title: Protected Runtime Artifacts - Soul File Injection Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Protected Runtime Artifacts - Soul File Injection Model.md"]
updated: 2026-07-24
---

# Protected Runtime Artifacts - Soul File Injection Model
## Purpose

Define how privileged runtime artifacts can shape Alice during execution without becoming ordinary repo-readable knowledge available through standard tooling surfaces.

This exists to solve one specific problem:

Some files must influence Alice at runtime.

Those same files must not become inspectable repo artifacts that Alice can read, search, summarize, or modify through ordinary workflows.

---

## Core Principle

A protected runtime artifact may be:

- runtime-visible
- repo-blind
- tool-blind
- non-editable by Alice

Alice may receive the effect of the artifact.

Alice may not receive ordinary reflective access to the artifact itself.

---

## Why This Exists

Normal repo files are visible through:

- file reads
- repo search
- code navigation
- diffs
- summaries
- indexing

This is acceptable for ordinary code.

It is not acceptable for identity-shaping artifacts such as:

- soul files
- protected runtime identity definitions
- core personality shaping artifacts
- privileged instruction layers

This model ensures:

**Alice can be shaped by an artifact without being able to inspect it as source material.**

---

## Definitions

### Protected Runtime Artifact

A file used to shape or constrain Alice at runtime that must not be exposed as normal repo content.

### Soul File

A special protected runtime artifact that defines or materially shapes Alice’s identity and behavior.

### Runtime-Visible

Accessible to the runtime loader so its effects can be applied.

### Repo-Blind

Not accessible to Alice through repo tools, search, diffs, or indexing.

---

## Core Distinction

Three visibility modes:

### 1. Repo-Readable

Alice can inspect normally.

### 2. Runtime-Consumed but Repo-Blind

Runtime can use it, Alice cannot inspect it.

### 3. Fully Blind

Alice cannot inspect and does not receive raw contents.

Soul files should generally be:

**Runtime-consumed but repo-blind**

---

## System Model

.evo_env/[soul.md](http://soul.md/)

↓

Protected Runtime Loader

↓

Soul Compiler

↓

Compiled Artifact (JSON or equivalent)

↓

Runtime Injection

↓

Alice Session

---

## Soul File Rule

Soul files are not ordinary prompts.

They are **protected runtime identity artifacts**.

Alice must not:

- read them
- search them
- inspect diffs
- summarize contents
- modify them

Alice may only experience the resulting runtime identity state.

---

## Runtime Injection Model

A protected loader:

- reads artifact from `.evo_env`
- compiles it into structured format
- injects resulting state into runtime
- prevents raw artifact exposure

---

## What Alice May Receive

Alice may receive:

- identity state
- behavior constraints
- tone/voice shaping
- runtime configuration derived from artifacts

Alice may not receive:

- raw file contents
- direct file access
- diff history
- editing authority

---

## Relationship to `.evo_env`

`.evo_env` stores protected runtime artifacts.

- runtime loaders may access it
- Alice may not

This creates:

- runtime visibility
- repo blindness

---

## Relationship to Delegator

Delegator governs interaction, not identity loading.

Delegator controls:

- whether workflows run
- which agents are used
- output constraints

Runtime injection controls identity shaping.

---

## Relationship to Repo Adapters

Repo adapters must treat protected runtime artifacts as:

- unreadable
- non-searchable
- non-diffable
- non-editable by Alice

Changes must go through protected workflows.

---

## Protected Change Rule

Protected runtime artifacts may only be modified through:

1. human-defined or approved task
2. scoped external agent or protected workflow
3. review artifact generation
4. human approval
5. runtime reload

---

## Metadata Leakage Rule

Minimize leakage through:

- filenames
- paths
- diffs
- logs
- summaries

---

## Technical Enforcement

Must be enforced at system level:

- hidden from file enumeration
- excluded from search
- excluded from indexing
- blocked at repo adapter level
- only accessible via protected loader

---

## System-Wide Scope

This applies across:

- editor
- repo adapter
- search/indexing
- runtime loader
- file system access

---

## User Authority

User may:

- inspect
- edit
- approve changes
- control visibility

---

## Principle

Do not make identity-defining artifacts ordinary readable repo knowledge.

---

## End State

A mature system supports:

- normal repo files
- protected blind zones
- runtime-consumed identity artifacts
- compiled runtime state
- human-controlled updates

---

## Summary

Protected runtime artifacts allow:

- runtime shaping of Alice
- separation of effect and inspectability
- strong control over identity layer

They prevent:

- Alice reading identity-defining artifacts
- self-modification risk
- over-reliance on governance

Runtime may consume the artifact.

Alice may not inspect it as ordinary code.

---

## Related Notes

- [EVO Blind Zone / .evo_env Protected Workspace Model](https://app.notion.com/p/342c72bad01381a4ad10c0da5891b0fc)
- Protected System Zones and Privileged Change Policy
- [Delegator — Execution Governance Doctrine](https://app.notion.com/p/342c72bad01381e088ecc512452813e4)
- Repo Adapter Design
^[{src_rel}]
