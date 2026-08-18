---
title: evoconnect-talent-backlink-navigation-doctrine
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/evoconnect-talent-backlink-navigation-doctrine.md"]
updated: 2026-07-24
---

# EVOconnect — Talent Backlink Navigation Doctrine

## Purpose

This raw doctrine note defines how Talents, Methods, Task Chains, and Talent Linked Notes use backlinks and traversal structure.

It extends the existing EVOconnect doctrine for:

- Talents,
- Methods,
- Task Chains,
- Delegator,
- Linked Notes,
- Living Notes,
- and Task Manager runtime state.

This note focuses specifically on:

- operational traversal,
- bounded execution navigation,
- lightweight workflow movement,
- and how Alice navigates executable workflow structures without loading excessive context.

This note does not replace Delegator manifest enforcement.

Backlinks guide traversal.

The manifest and Delegator govern executable authority.

---

## Core Principle

Backlinks inside Talent structures are traversal rails, not generic wiki links.

Talent traversal must remain:

- lightweight,
- bounded,
- resumable,
- deterministic,
- and operationally understandable.

A Talent should not require loading the entire workflow package to execute a single step.

---

## Cognitive Layer Relationship

The EVO cognition system separates:

### Linked Notes

Linked Notes provide canonical context.

They represent:

- durable knowledge,
- architecture,
- concepts,
- relationships,
- references,
- and long-term connected understanding.

### Journals

Journals help Alice learn.

They represent:

- reflection,
- interpretation,
- adaptation,
- behavioral learning,
- and user understanding.

### Talent Linked Notes

Talent Linked Notes show Alice how to act.

They represent:

- operational workflow structure,
- bounded procedures,
- executable traversal,
- reusable execution guidance,
- and governed operational memory.

### Task Manager Artifacts

Task Manager artifacts track what actually happened during execution.

They represent:

- runtime state,
- completed steps,
- checkpoints,
- outputs,
- approvals,
- verification,
- failures,
- and resumable execution state.

---

## Talent Linked Notes

Talent Linked Notes are a distinct note category.

They exist between:

- journal entries,
- and Living Notes.

Talent Linked Notes are:

- co-owned by the user and Alice,
- readable by the user,
- traversable by Alice,
- operationally structured,
- versioned,
- hashable,
- and governed by Method/Talent revision rules.

Talent Linked Notes are not normal editable Living Notes.

Users may inspect them but may not directly edit them manually.

Alice may modify them only through approved Method or Talent revision flows.

Changes may:

- create a new version,
- invalidate trust,
- invalidate hashes,
- reset promotion state,
- or require re-verification.

---

## UX Model

Living Notes should appear:

- clean,
- structured,
- Obsidian-like,
- readable,
- connected,
- and concept-oriented.

Talent Linked Notes should appear operational.

User-facing Talent structures should resemble:

```text
Talent
  Step 01
  Step 02
  Step 02a
  Step 03
```

Task Chains should resemble:

```text
Task Chain
  Talent A
    Step 01
    Step 02
  Talent B
    Step 01
    Step 02
```

Talent Linked Notes use the same linking surface as task/project notes, but visually present as executable workflow hierarchy.

---

## Traversal Hierarchy

Traversal hierarchy is strict.

```text
Task Chain
  → Talent
    → Step
```

Rules:

- Task Chains traverse Talents.
- Talents traverse steps.
- Steps do not directly traverse Task Chains.
- Steps should not create arbitrary cross-chain traversal.
- Variation steps remain inside the parent Talent.

This prevents traversal chaos and preserves bounded execution structure.

---

## Lean Traversal Rule

Traversal must remain lean.

Alice should not load:

- the entire Task Chain,
- the entire Talent package,
- all previous execution history,
- or all linked doctrine notes

just to execute one step.

A step should execute using only:

1. Parent Talent summary.
2. Current step note.
3. Current runtime artifact state when needed.
4. Previous checkpoint/output when needed.
5. Manifest route and allowed tools.
6. Explicitly linked required context.

If a step requires loading the entire workflow history to understand it, the step is too large or too poorly linked.

---

## Step Locality Rule

A step must contain enough local context to execute safely.

A step may reference:

- parent Talent,
- previous step artifact,
- current run artifact,
- approved linked context,
- required deliverables,
- and explicitly allowed references.

Traversal should remain bounded.

Step notes should not become giant embedded workflow documents.

---

## Required Step Navigation Metadata

Every canonical step note must include traversal metadata.

Minimum navigation metadata:

```yaml
parent_talent:
previous_step:
next_step:
variation_of:
```

Rules:

- `parent_talent` is always required.
- `previous_step` is optional only for the first step.
- `next_step` is optional only for the final step.
- `variation_of` exists only for variation steps.
- Variation steps still participate in normal traversal order.
- Parent numeric steps may link to known variations as optional branches.

Example:

```yaml
parent_talent: [[Talent — Issue Closeout]]
previous_step: [[01-load-issue-set]]
next_step: [[03-compare-planning-spec]]
variation_of: [[02-map-code-evidence]]
```

---

## Talent Navigation Rules

The parent Talent note acts as the operational route overview.

The parent Talent note should:

- summarize the workflow,
- summarize deliverables,
- summarize permissions and allowed tools,
- link to step notes in route order,
- link to known variation routes,
- link to required context,
- and link to related Task Chains when applicable.

The Talent note is the operational entry point.

---

## Variation Traversal Rules

Variation steps are alternate traversal nodes.

Variation steps:

- replace a parent numeric step,
- remain inside the same Talent,
- preserve normal traversal links,
- and must link to the parent step they replace.

Parent numeric steps may link to known variations as optional execution paths.

Variation traversal must remain explicit.

Alice must not improvise undeclared variation traversal.

Delegator manifest enforcement still governs which route is executable.

---

## Backlinks Versus Manifest Authority

Backlinks help Alice navigate workflow structure.

Backlinks are not executable authority.

The Delegator and manifest still define:

- allowed execution routes,
- allowed transitions,
- allowed tools,
- approvals,
- trusted paths,
- and executable authority.

Alice may follow backlinks for understanding and traversal.

Delegator determines whether a traversal path is executable.

---

## Runtime State Separation

Talent Linked Notes are not runtime state.

Task Manager artifacts are runtime state.

Task Manager artifacts should track:

- active run,
- current step,
- completed steps,
- checkpoints,
- outputs,
- approval state,
- verification state,
- failures,
- retries,
- and resumability.

Talent Linked Notes answer:

```text
Where do I go next?
```

Task Manager artifacts answer:

```text
Where am I in this run, what completed, and what output was produced?
```

---

## External-Agent Simulation

External agents without EVO Task Manager integration may simulate runtime state using markdown artifact files.

Example:

```text
artifacts/current-run.md
```

This simulates Task Manager runtime tracking.

Inside EVO proper, the Task Manager is the authoritative runtime state ledger.

Markdown runtime artifacts are optional debug/export representations inside the EVO app.

---

## Resumability

Backlink traversal supports resumable execution.

Alice should always be able to determine:

- current route,
- current step,
- previous checkpoint,
- next allowed step,
- and required deliverables

without reconstructing the entire workflow from chat history.

Traversal should rely on:

- Talent Linked Notes,
- manifests,
- and Task Manager artifacts

instead of giant conversational memory replay.

---

## Canonical Traversal Rule

A Talent or Task Chain is not canonically ready until traversal works cleanly.

Traversal is considered valid when:

- Alice can enter from the parent node,
- follow deterministic operational links,
- execute bounded local steps,
- return to the parent route,
- determine the next step,
- and resume after interruption using Task Manager state.

---

## Summary

Backlinks inside Talent structures are operational traversal rails.

Linked Notes provide canonical context.

Journals help Alice learn.

Talent Linked Notes show Alice how to act.

Task Manager artifacts track what actually happened.

Task Chains traverse Talents.

Talents traverse steps.

Traversal must remain lean, bounded, resumable, and operationally understandable.

Backlinks help Alice navigate.

Delegator and manifests determine executable authority.

## Related

^[{src_rel}]
