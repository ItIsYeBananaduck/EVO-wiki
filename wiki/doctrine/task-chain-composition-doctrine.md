---
title: task-chain-composition-doctrine
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/task-chain-composition-doctrine.md
updated: 2026-07-24
---

# Task-Chain Composition Doctrine

## Purpose

Task-chain composition defines how EVO workflow skills invoke supporting skills without loading every available skill into the agent context. It keeps execution deterministic, token-light, and auditable by making nested skill boundaries explicit in each workflow's own files.

This doctrine applies to EVO workflow skills such as `evo-plan`, `evo-run`, `evo-analyze`, and `evo-closeout`.

## What Task-Chain Composition Is

Each EVO skill declares exactly the nested skills it may use through `linked-skills/` descriptors. The task chain selects those supporting skills explicitly, by step or phase, before execution reaches the relevant boundary.

Skill selection is bounded. A workflow does not infer supporting skills from a full inventory scan while it is already executing. It uses the descriptors attached to its own chain.

Task chains compose only the skills required for the workflow. Unused skills are not loaded, summarized, or carried as context.

## What Task-Chain Composition Is Not

Task-chain composition is not global skill injection. Agents must not inject all available skills into context simultaneously.

It is not runtime skill discovery. Agents should not browse `SKILLS_INVENTORY.md` during execution to guess which supporting skill to load. `SKILLS_INVENTORY.md` is for discovery before choosing a workflow, not for expanding a running chain opportunistically.

It is not a monolithic super-prompt. The point is to keep each workflow small enough to reason about, while still making its allowed supporting skills explicit.

## Linked-Skills Descriptor Pattern

Descriptor files live under:

```text
.codex/skills/<evo-skill>/linked-skills/
```

Each nested supporting skill gets one descriptor file. The descriptor must declare:

- source path
- allowed steps or phases
- purpose
- allowed scope
- forbidden scope
- staleness rules, when the supporting skill depends on indexed or external state
- invocation note, including whether the skill is mandatory or conditional and what trigger activates it

`chain.md` must include a `Linked-Skill Invocation Boundaries` table that names the skill, allowed steps, condition, and descriptor path.

Use `.codex/skills/evo-plan/linked-skills/` as the format reference. `evo-run`, `evo-analyze`, and `evo-closeout` follow the same pattern.

## Token Discipline

Nested skills are invoked only when the task chain requires them.

GitNexus usage must stay targeted. Query specific symbols, files, routes, or concepts. Do not dump the full index into an EVO workflow.

Caveman-lite is the default communication posture for EVO workflow status updates. Generated artifacts, blocker reports, doctrine conflict findings, safety warnings, and final reports remain fully written.

## Authoring New EVO Skills

New EVO skills that invoke supporting skills must include a `linked-skills/` directory.

Each supporting skill invocation must have a descriptor file before the workflow depends on it.

The skill's `chain.md` must include a `Linked-Skill Invocation Boundaries` table and must keep any supplementary prose subordinate to that table.

If a supporting skill has stale-state risk, network-state risk, indexed-graph risk, or external-authority risk, the descriptor must include the stop or warning rule that governs that risk.

## Related Doctrine

- [[EVE Governance MOC]]
- [[EVOconnect — Skill Import and Conversion Doctrine]]
- [[EVO — Talent Tool Envelope & Context Compression Doctrine]]
^[source-materials/mirrors/doctrine/task-chain-composition-doctrine.md]
