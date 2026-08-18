---
title: repo-instruction-routing
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/repo-instruction-routing.md"]
updated: 2026-07-24
---

# Repo Instruction Routing Planning Spec

## Purpose

Define a repo-native instruction routing system that helps agents orient themselves before acting.

The goal is to reduce architecture drift by making the repository self-describing.

Agents should not rely on stale memory, old issue wording, legacy folder structure, or outdated implementation patterns when doing work.

Before implementation, agents must traverse the repo routing layer, load the relevant folder instructions, check linked doctrine, and surface conflicts before proceeding.

---

## Problem

Agents repeatedly drift toward stale assumptions, including:

- treating Supabase as runtime truth,
- treating the SvelteKit app as the primary runtime surface,
- missing that EVO is on-device-first,
- forgetting that Flutter is the active implementation surface,
- treating Coach as a separate product truth instead of a desktop training surface,
- missing that EVOtraining and Coach can share logic but diverge in UI,
- failing to recognize where canonical shared assets live,
- reusing deprecated Alice avatar assets,
- and forgetting current Talent/runtime architecture changes.

This happens because agents load broad context inconsistently and do not have a deterministic repo traversal requirement.

---

## Core Principle

The repo should act as a self-describing execution environment.

Top-level agent instructions define agent posture and repo nonnegotiables.

A universal routing document tells agents where to find local instructions.

Folder-level instruction files provide concise orientation and anti-drift rules.

Skills and workflows perform repeatable execution.

Runtime artifacts preserve active execution state.

---

## Agent Posture Model

All agents share repo truth, but they do not share the same operational posture.

### Claude

Claude is the analyst/planner agent.

Claude should focus on:

- architecture analysis,
- doctrine reconciliation,
- planning,
- issue shaping,
- dependency analysis,
- conflict detection,
- and execution prompt preparation.

Claude may propose doctrine or planning changes, but should avoid implementation unless explicitly instructed.

### Workers

Workers include Codex, Gemini, Copilot-like agents, and similar implementation agents.

Workers should focus on:

- scoped implementation,
- tests,
- validation,
- PR preparation,
- Linear evidence,
- and issue completion.

Workers must not redesign architecture or expand doctrine unless explicitly instructed.

### Windsurf

Windsurf is a local adaptable IDE operator.

Windsurf may remain locally customized and flexible.

Windsurf should still respect repo truth when editing repo files, but its local rules may remain user-controlled and gitignored.

---

## Routing Flow

The intended traversal flow is:

```text
agent-local instructions
→ .evo/routing.md
→ nearest relevant INSTRUCTIONS.md
→ linked doctrine / planning specs
→ applicable skill or workflow
→ runtime artifact
```

Agents must not skip directly from issue text to implementation when folder instructions or doctrine may alter the correct behavior.

---

## Top-Level Instruction Contract

Top-level instruction files should explain:

- the agent’s role in the repo,
- what the repo is as a whole,
- current nonnegotiables,
- the source-of-truth hierarchy,
- the traversal mandate,
- conflict handling rules,
- and where to load the routing map.

Top-level instructions should not duplicate all doctrine.

They should point to the routing system and require agents to traverse it.

---

## Universal Routing Map

The canonical routing map should live at:

```text
.evo/routing.md
```

This file should define:

- traversal rules,
- repo map,
- instruction discovery rules,
- source-of-truth hierarchy,
- conflict handling rules,
- workflow/skill discovery rules,
- and stop conditions.

It should not become a giant doctrine dump.

---

## Folder Instruction Model

Folder-level instruction files should be named:

```text
INSTRUCTIONS.md
```

These files should be concise orientation and anti-drift boundaries.

They should answer:

- what this folder is,
- what it owns,
- what is canonical here,
- what must not drift,
- what doctrine or specs are relevant,
- what workflows apply,
- and when agents must stop.

They should not become large doctrine documents.

---

## Proposed INSTRUCTIONS.md Template

```md
# Purpose

What this folder contains and why it exists.

# Source of Truth

What owns truth for this area.

# Local Rules

Rules agents must follow when working here.

# Relevant Doctrine

Repo-relative links to doctrine or planning specs.

# Workflows

Skills or workflows that apply in this area.

# Stop Conditions

When agents must stop, report a conflict, or ask for planning before implementation.
```

---

## First-Wave Routing Boundaries

The first routing pass should cover only major drift-prone folders.

### Root

The repo root owns repo identity, agent posture, nonnegotiables, and traversal mandate.

### docs/

Documentation boundary.

One instruction file is likely enough because `docs-ingest` owns detailed lifecycle behavior.

Expected rules:

- use docs-ingest for doc placement/normalization,
- `docs/raw` contains exploratory concepts,
- `docs/planning-spec` contains implementation planning specs,
- `docs/EVOnotes` contains durable notes/doctrine,
- no Notion links in YAML/frontmatter graph fields,
- do not promote raw concepts to doctrine without review.

### flutter_app/

Shared Flutter implementation root for EVOtraining and Coach.

Flutter is the active primary app implementation surface.

EVOtraining and Coach may share business logic but do not necessarily share UI.

This folder should route to platform/product surface instructions.

### flutter_app/ios/

EVOtraining iOS surface.

Mobile EVOtraining user runtime.

Should link primarily to training doctrine.

### flutter_app/android/

EVOtraining Android surface.

Initially may point to `flutter_app/ios/INSTRUCTIONS.md` until Android-specific divergence exists.

Rule: do not fork platform instructions until divergence is real.

### flutter_app/macos/

Coach Mac surface.

Coach is a desktop training-management surface sharing logic with EVOtraining.

Coach is not a separate architectural truth source.

Coach may have specific doctrine links, but should mostly route to:

```text
docs/EVOnotes/doctrine/training/
```

### flutter_app/windows/

Coach Windows surface.

Initially may point to `flutter_app/macos/INSTRUCTIONS.md` until Windows-specific divergence exists.

Rule: do not fork platform instructions until divergence is real.

### app/

SvelteKit/web surface.

Purpose:

- EVOsystem/EVOtraining marketing site,
- online marketplace surfaces,
- web-facing informational/admin surfaces,
- routing/linking toward EVOtraining.app.

Nonnegotiables:

- this is not the primary runtime architecture,
- Flutter remains the canonical active runtime/app surface,
- do not reintroduce web-first runtime assumptions,
- do not migrate core runtime truth into web infrastructure.

### packages/

Shared assets and shared infrastructure inventory.

This instruction file should explain what shared systems/assets exist and what they are used for.

Important rule:

- canonical Alice avatar assets/shared avatar UI belong in shared packages unless platform-specific rendering requires otherwise,
- deprecated Alice avatar assets must not be reused,
- app-local avatar duplication should be treated as cleanup/migration candidate,
- shared logic used by EVOtraining and Coach should live here where appropriate.

### supabase/

Limited cloud/service integration boundary.

Allowed roles may include:

- auth,
- subscriptions,
- marketplace/payment support,
- trainer/client relationship metadata where explicitly approved,
- optional cloud continuity/sync support,
- global adapter/delta aggregation if doctrine allows it.

Nonnegotiables:

- Supabase is not primary runtime truth,
- existing tables are not proof of current architecture,
- legacy/transitional tables must not be treated as canonical without doctrine confirmation,
- do not build new runtime truth on stale tables.

### .github/

Repository automation and governance workflows.

Contains:

- GitHub Actions,
- PR automation,
- CI/CD behavior,
- review automation integrations.

Rules:

- do not modify workflows casually,
- CI changes require explicit validation,
- repo automation changes affect all agents/contributors,
- branch governance must align with current repo doctrine,
- CodeRabbit/CI automation changes are high-impact.

### .codex/

Codex worker execution surface and canonical shared executable skill source.

Discovery:

- Codex does not play nicely with symlinked skills.
- Therefore `.codex/skills/` is currently the canonical runnable skill source of truth.
- Other agents may link/reference Codex skill content, but Codex owns the runnable skill layout.

Rules:

- do not symlink Codex skills,
- do not duplicate shared skill behavior without a clear adapter reason,
- preserve Codex-compatible skill layout.

### .claude/

Claude analyst/planner overlays, worktrees, and Claude-specific behavior.

Claude should use this area for analysis/planning context and Claude-specific skills or overlays.

Claude is not the canonical shared executable skill source.

---

## Conflict Handling

If traversal finds a conflict between:

- issue wording,
- local folder instructions,
- doctrine,
- code structure,
- or existing database/schema artifacts,

then the agent must stop and surface the conflict before implementation.

Agents must not silently implement stale architecture.

Expected conflict report:

```text
I found an architecture conflict before implementation.
Stale assumption:
Current doctrine/instruction:
Affected files/issues:
Recommended correction:
```

---

## Additions Rule

Any new significant top-level folder, major feature folder, shared package, platform surface, doctrine folder, or agent workflow area must either:

1. include an `INSTRUCTIONS.md`, or
2. be explicitly covered by a parent routing entry.

This prevents future unrouted architecture drift.

---

## Skill Relationship

Folder instructions orient agents.

Skills perform repeatable workflows.

Instructions should not duplicate full skill logic.

Current practical skill source rule:

```text
.codex/skills/ = canonical runnable shared skill source for now
```

This is due to Codex symlink limitations.

Future work may revisit shared skill routing after compatibility is proven.

---

## Initial Implementation Goal

The first implementation pass should create only:

- top-level routing support,
- first-wave `INSTRUCTIONS.md` files,
- concise anti-drift rules,
- and references to relevant doctrine/paths.

It should not perform broad repo refactors.

It should not rewrite code.

It should not normalize all docs.

It should not migrate skills beyond the currently agreed Codex-source-of-truth model.

---

## Open Questions For evo-plan

- Should `.evo/routing.md` be introduced before folder-level `INSTRUCTIONS.md`, or in the same cluster?
- Should `AGENTS.md` and `CLAUDE.md` be updated in the same cluster or separate clusters?
- Should first-wave folder instructions be one parent cluster, or split by docs/app/platform/agent areas?
- Should `docs-ingest` gain validation support for missing or stale `INSTRUCTIONS.md` in this same cluster or a follow-up cluster?
- Should package asset inventory be manually audited before writing `packages/INSTRUCTIONS.md`?
- Should `.codex/skills/` canonical status be documented only, or also enforced through tests/checks?

## Related

^[{src_rel}]
