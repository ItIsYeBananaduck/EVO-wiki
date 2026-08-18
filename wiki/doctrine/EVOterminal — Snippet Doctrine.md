---
title: EVOterminal — Snippet Doctrine
type: concept
tags: [doctrine, evo, terminal]
sources:
  - source-materials/mirrors/doctrine/EVOterminal — Snippet Doctrine.md
updated: 2026-07-23
---
# EVOterminal — Snippet Doctrine

## Purpose

Snippets are reusable, user-approved terminal command templates inside EVOterminal.

Their primary purpose is to:
- reduce repetitive typing
- reduce token usage for Alice
- create safer terminal execution flows
- provide reusable building blocks for Talents and Methods
- help non-terminal-native users safely operate advanced workflows

Snippets are intentionally positioned between:
- raw terminal access
- full Talent automation

They are lightweight execution primitives.

---

# Core Philosophy

A snippet is not arbitrary code execution.

A snippet is:
- a known command
- with known intent
- optional parameters
- known permission level
- known execution scope
- known safety classification

This allows Delegator to reason about execution safely.

---

# User Experience

Users can:
- save commonly used commands
- categorize snippets
- favorite snippets
- inject snippets directly into EVOterminal
- optionally auto-execute safe snippets
- share snippets into Methods and Talents

Primary UX flow:

1. User selects snippet
2. Snippet injects into terminal
3. User reviews
4. User executes

For safe snippets:

1. User selects snippet
2. Delegator validates permissions
3. Snippet executes directly

---

# Snippet Categories

## Safe Snippets

Safe snippets may:
- read files
- navigate directories
- run non-destructive package commands
- perform diagnostics
- run approved development workflows

Examples:

```bash
git status
flutter pub get
npm install
ls
pwd
```

Safe snippets may be:
- one-click executable
- reusable by Talents without additional approval
- used during unattended workflows if previously approved

---

## Restricted Snippets

Restricted snippets may:
- mutate files
- delete content
- alter permissions
- access system-level resources
- run scripts
- invoke package managers with elevated risk

Examples:

```bash
rm -rf
chmod
sudo commands
git reset --hard
```

Restricted snippets require:
- explicit approval
- Delegator validation
- optional secondary confirmation

Alice may propose them.

Alice may not silently execute them.

---

# Delegator Integration

Every snippet must:
- register capability metadata
- declare execution class
- declare permission level
- declare tool requirements
- declare risk profile

Delegator should:
- inspect snippet metadata
- determine if approval is required
- block unsafe parameter injection
- prevent unauthorized escalation
- audit executions

Snippets become governed execution objects.

---

# Talent Integration

Talents may reference snippets directly.

Example:

- Talent calls `git_status`
- Delegator resolves snippet
- Terminal executes canonical command

Benefits:
- lower token usage
- deterministic execution
- reduced hallucination
- consistent workflows
- reusable execution primitives

This also allows Alice to learn workflows by:
- composing snippets
- sequencing snippets
- promoting snippet chains into Methods
- promoting Methods into Talents

---

# Parameterized Snippets

Snippets may optionally support parameters.

Example:

```bash
git checkout {{branch}}
```

Delegator must:
- sanitize parameters
- validate allowed argument patterns
- prevent command injection

Parameterized snippets should remain constrained.

---

# Snippet Governance

Snippets are user-owned.

Users control:
- visibility
- execution permissions
- unattended execution eligibility
- Talent accessibility
- deletion

System snippets may also exist.

System snippets are:
- app-provided
- immutable
- versioned
- reviewed

---

# Relationship to Methods

Snippets are not Methods.

Snippets are low-level execution units.

Methods are:
- ordered workflows
- reasoning-aware
- multi-step
- governance-aware

A Method may:
- use multiple snippets
- include browser steps
- include terminal steps
- include approval gates

Talents may ultimately compile down into snippet execution chains.

---

# Relationship to Session Recipes

Session recipes define terminal startup behavior.

Examples:
- human-readable name
- working directory
- startup command
- environment or context label
- repo context
- branch context
- Linear issue or cluster context
- fresh vs resumed session behavior
- safety and approval requirements

Session recipes may invoke snippets.

Snippets represent executable operational primitives.

Recipes represent session launch context. A recipe can call a snippet during startup, but a snippet should not own working-directory, branch, Linear issue, or fresh/resume semantics.

---

# Cross-Device Execution

Snippets must support cross-device orchestration.

A snippet should be executable from:
- desktop
- mobile
- remote Hive node
- delegated execution environment

Execution state must persist independently from conversational memory.

---

# Token Efficiency

Snippets are one of the primary token-efficiency mechanisms inside Connect.

Without snippets:
- repeated workflows require repeated inference

With snippets:
- Alice invokes known workflows directly

This reduces:
- reasoning overhead
- context growth
- execution ambiguity
- operational drift

---

# Long-Term Vision

Over time:
- Alice learns workflows
- workflows become snippets
- snippets become Methods
- Methods become Talents
- Talents reduce dependence on external agents

Snippets are one of the foundational mechanisms that allow Connect to evolve from:
- AI-assisted terminal interaction

into:
- governed orchestration infrastructure.

## Related
- [[EVO Architecture Bible]]
- [[EVOterminal — Session Recipe Doctrine.md]]
^[source-materials/mirrors/doctrine/EVOterminal — Snippet Doctrine.md]
