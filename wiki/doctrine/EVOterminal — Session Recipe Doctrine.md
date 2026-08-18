---
title: EVOterminal — Session Recipe Doctrine
type: concept
tags: [doctrine, evo, terminal]
sources:
  - source-materials/mirrors/doctrine/EVOterminal — Session Recipe Doctrine.md
updated: 2026-07-23
---
# EVOterminal — Session Recipe Doctrine

## Purpose

Session recipes describe governed terminal startup workflows.

They let Alice convert user intent into a reviewable launch plan instead of making the user remember a terminal profile, shell command, working directory, branch, or issue context.

Session recipes are the semantic profile layer for EVOterminal.

---

## Recipe Shape

A session recipe should represent:

- human-readable name
- working directory
- startup command
- environment or context label
- repo context
- branch context when known
- Linear issue or cluster context when known
- fresh-session or resumed-session mode
- safety and approval requirements
- optional snippets invoked during startup

Recipes are not raw shell history. They are structured launch intents that Delegator can inspect.

---

## Alice-Created Recipe Flow

Alice may create a recipe from a user request.

Example request:

> Start EVOterminal in the GitFit repo and invoke Codex.

Resolution:

1. Identify the target environment.
2. Resolve the working directory.
3. Resolve repo and branch context.
4. Select a startup command.
5. Determine whether the session must be fresh or may resume.
6. Attach Linear issue or cluster context when present.
7. Present a recipe preview before launch.

The user approves the resolved recipe before first execution unless policy allows direct launch.

---

## Fresh vs Resumed Sessions

The UI must distinguish fresh and resumed sessions before launch.

Fresh sessions:

- start without inherited terminal state
- are preferred for bounded Codex runs
- are required for governed cluster execution unless the user explicitly resumes the same cluster
- prevent hidden carryover from stale commands or environment state

Resumed sessions:

- continue an explicitly selected prior workflow
- preserve context only when the prior session is still valid
- must show the source session or workflow being resumed
- should not be inferred silently for governed work

Launch controls should make this distinction visible, not implicit.

---

## Relationship to Snippets

Session recipes define how EVOterminal starts.

Snippets define reusable commands or command templates inside a session.

A recipe may invoke one or more snippets as part of startup, but snippets do not replace recipes. Recipes own session context, launch mode, and approval requirements. Snippets own executable primitives.

---

## Codex Bounded-Session Workflows

Codex workflows are a canonical recipe family.

Useful recipes include:

- open a repo and start Codex
- start a fresh governed EVO cluster session
- continue a named Linear cluster from Linear state
- run a known validation command
- open an SSH workflow with explicit approval

For governed EVO cluster work, the recipe should carry:

- parent issue identifier
- target branch
- parent-scoped branch name
- fresh-session requirement
- validation command guidance
- no-push or PR-delivery rules when present

The recipe does not replace Linear as execution truth. It only launches the governed environment with the correct context.

---

## Governance

Delegator should inspect recipes before launch.

Approval requirements should consider:

- command risk
- working directory scope
- repo trust
- branch mutation
- remote or SSH access
- whether the recipe writes files
- whether the recipe can push, deploy, or create external side effects

Approved recipes may support one-click launch, but only within their declared scope.

---

## UI Placement

Desktop Command Center should expose session recipes alongside EVOterminal.

The surface should include:

- saved recipes
- recent workflows promoted into recipes
- Alice-created recipe preview
- launch mode indicator
- approval status
- repo, branch, and Linear context chips

Control Center may expose favorites or recent safe recipes, but detailed preview and approval belong in Command Center or a focused launch sheet.

## Related
- [[EVO Architecture Bible]]
- [[EVOterminal — Snippet Doctrine.md]]
^[source-materials/mirrors/doctrine/EVOterminal — Session Recipe Doctrine.md]
