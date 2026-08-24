---
title: EVO Blind Zone - .evo_env Protected Workspace Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVO Blind Zone - .evo_env Protected Workspace Model.md"]
updated: 2026-07-24
---

# EVO Blind Zone - .evo_env Protected Workspace Model
## Purpose

Define a protected workspace boundary for code and files that Alice must never be able to inspect directly.

This exists to solve one specific problem:

Alice may safely coordinate work in many areas.

Alice must not be able to see or directly reason over the parts of the system that materially govern, define, or constrain Alice herself.

---

## Core Principle

.evo_env is not just a folder.

It is a **protected blind zone**.

Anything inside .evo_env is permanently hidden from Alice.

---

## Why This Exists

Normal governance solves most problems:

- Alice proposes
- Delegator governs
- system executes

That is not sufficient for:

**self-referential code access**

If Alice can inspect the exact code that defines or constrains her, the system relies too heavily on governance alone.

.evo_env introduces a stronger guarantee:

**Alice cannot misuse what she cannot see.**

---

## Core Rule

Alice must never be able to:

- read files inside .evo_env
- search inside .evo_env
- index .evo_env content
- summarize .evo_env content
- inspect diffs from .evo_env
- infer contents through tooling surfaces

This must be enforced technically, not through instruction.

---

## Intended Use

.evo_env is for privileged code or artifacts such as:

- Alice runtime shaping code
- identity-sensitive logic
- governance-sensitive implementation
- protected execution logic
- self-modification risk areas
- experimental protected system behavior

Primary use case: git-fit repository.

---

## Visibility Model

### Alice

Alice has **no visibility** into .evo_env.

- no file reads
- no search results
- no embeddings
- no diffs
- no summaries

Alice may know the zone exists, but not what is inside.

---

### External Agents

External agents may be granted scoped access.

Constraints:

- path-scoped
- task-scoped
- non-sovereign
- reviewable

They are workers, not authorities.

---

### Human User

The user has full authority:

- inspect contents
- define tasks
- approve prompt relay
- review outputs
- approve merges

---

## Delegator Role

Delegator governs interaction with the blind zone.

It decides:

- whether a task may be initiated
- which agent may access the zone
- whether prompt relay must be exact
- whether transformation is allowed
- what output may exit the zone
- whether human review is required

Delegator does not expose .evo_env contents to Alice.

---

## Prompt Relay Model

### Default: Exact Relay Mode

- user provides instruction
- Alice relays it exactly
- no reinterpretation
- no hidden modification

### Optional: Approved Transform Mode

- Alice may structure prompt
- requires explicit approval

Protected zones should default to **Exact Relay Mode**.

---

## Output Model

Outputs must be constrained.

### Allowed Types

**Status Only**

- started
- completed
- failed

**Review Artifact**

- patch
- branch
- PR

**Human Summary**

- only if explicitly allowed

**Human Escrow**

- visible only to user

### Default for .evo_env

**Review Artifact + Human Review Required**

---

## Merge / Apply Rule

Alice may not directly merge or apply changes from .evo_env.

All changes must require human review.

---

## Metadata Leakage Constraint

System should minimize leakage through:

- filenames
- paths
- diffs
- logs
- stack traces
- summaries

Perfect isolation is not required, but leakage must be treated as a real risk.

---

## Technical Enforcement

Blindness must be enforced at the system level:

- file access blocked
- search blocked
- indexing excluded
- diff rendering blocked
- adapter-level path denial

If not technically enforced, it is not a real blind zone.

---

## Relationship to Repo Adapters

Normal repo:

- Alice-readable
- governed writes

.evo_env:

- not readable by Alice
- external-agent access only
- human-reviewed output

This is a **policy enclave**, not a separate repo model.

---

## Relationship to Delegator

Delegator = governance

.evo_env = containment

Together they provide:

- execution control
- self-reference isolation

---

## Non-Negotiable Rule

Alice may coordinate work involving .evo_env.

Alice may never see inside .evo_env.

---

## Principle

Do not rely solely on alignment when structural blindness is possible.

---

## End State

A mature EVO system supports:

- governed normal repo workflows
- protected blind zones
- exact prompt relay
- constrained outputs
- human-reviewed protected changes

---

## Summary

.evo_env is a protected blind workspace.

It allows:

- safe coordination of privileged code work
- separation of authority and visibility
- protection against self-referential risk

It prevents:

- Alice inspecting privileged system logic
- reliance on governance alone
- unsafe self-modification paths

---

## Related Notes

- [Delegator — Execution Governance Doctrine](https://app.notion.com/p/342c72bad01381e088ecc512452813e4)
- EVOconnect — Canonical Definition
- [Data Sovereignty Doctrine](https://app.notion.com/p/33dc72bad01381eb9b4de2609f604c80)
- [Alice Identity Doctrine](https://app.notion.com/p/33dc72bad013811da04accd3f90303d3)
- Repo Adapter Design
- Protected System Zones and Privileged Change Policy
