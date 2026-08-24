---
title: Connect - UI Layer (Desktop)
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Connect - UI Layer (Desktop).md"]
updated: 2026-07-24
---

# Connect - UI Layer (Desktop)
[Connect - UI Layer (Desktop)](https://www.notion.so/33ec72bad0138134b877c5f10bc07505)
Desktop Connect = modular OS layer.
Features: - Slide-down panel or summonable overlay - Node status at a glance - My Tasks / Alice Tasks counters - Quick chat - Expandable deep mode - Internal browser - Internal terminal sandbox - EVOterminal session recipes
Visual Identity: - System-native feel - Subtle purple glow (Alice presence) - Blue swarm pulse for activity

## EVOterminal Session Recipes

Session recipes are a first-class Command Center surface for launching governed terminal workflows from intent.

They sit above static terminal profiles. Alice maps a user request such as "Start EVOterminal in the GitFit repo and invoke Codex" into a recipe preview that the user can review, approve, and launch.

The desktop UI should expose:

- saved session recipes
- recent terminal workflows
- Alice-created recipe previews before launch
- one-click launch for approved recipes
- fresh-session and resumed-session states as visibly distinct modes
- optional Linear issue, cluster, repo, and branch associations

Each recipe preview should show:

- human-readable name
- working directory
- startup command
- environment or context label
- repo context
- branch context when known
- Linear issue or cluster context when known
- whether the launch must be fresh or may resume
- approval and safety requirements

Fresh sessions are for bounded workflows that must start clean, such as a new Codex cluster run. Resumed sessions are for returning to an existing workflow whose state is still valid. The UI must not hide this distinction behind a generic launch button.

Alice may draft recipes from intent, recent terminal history, snippets, Methods, or Talents. The user still sees the resolved command, context, launch mode, and required approvals before execution unless the recipe has already been approved for one-click launch.

Codex bounded-session workflows are canonical examples:

- open the GitFit repo
- start a fresh Codex session
- run a governed EVO cluster
- run a known validation command
- resume an explicitly selected previous session

#connect

## Related
