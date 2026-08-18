---
title: EVOconnect — Lightweight Talent Structure Addendum
type: concept
tags: [connect, evo, talent]
sources:
  - source-materials/mirrors/doctrine/EVOconnect — Lightweight Talent Structure Addendum.md
updated: 2026-07-23
---
# EVOconnect — Lightweight Talent Structure Addendum

## Purpose

This note extends the existing Talent, Method, Delegator, and Task Chain doctrine.

It defines:

- lightweight Talent package structure,
- Method proving behavior,
- controlled variation routing,
- user-created Talent storage rules,
- and the separation between Task Chains, Methods, and Talents.

This note does not replace:
- Talent Definition,
- EVOconnect Talent Model,
- EVOconnect Method Specification Model,
- or Delegator doctrine.

It clarifies the structure and lifecycle of user-created Talents.

Related doctrine:

- [[EVOconnect — Delegator Talent Verification Doctrine]]
- [[MOC EVOconnect — Methods & Talents]]
- [[MOC EVOconnect — Delegator]]
- [[Talent Definition]]

---

## Core Principle

Talents must remain:

- lightweight,
- repeatable,
- understandable,
- auditable,
- and safe.

Talents are not arbitrary automation scripts.

Talents are governed reusable execution routes.

---

## Distinction Between Task Chains, Methods, and Talents

### Task Chains

Task Chains are orchestration structures.

Task Chains may organize:
- tasks,
- Methods,
- Talents,
- or mixtures of these.

Task Chains are not Talents.

Task Chains preserve higher-level workflow structure.

Task Chains may call:
- existing Talents,
- existing Methods,
- or generate new Method candidates.

---

### Methods

Methods are supervised proving workflows.

Methods exist primarily inside:
- task history,
- task chains,
- and active supervised execution.

Methods are not permanent Talent package contents by default.

Methods become Talents only after:
- Delegator validation,
- successful supervised execution,
- and promotion approval.

Methods may originate from:
- conversational teaching,
- explicit workflow observation,
- task/task-chain inference,
- imported skill conversion,
- or Talent variation proposals.

---

### Talents

Talents are trusted reusable execution routes.

A Talent is:
- repeatable,
- governed,
- bounded,
- and approved for autonomous reuse.

Talents are promoted from Methods after:
- 3 successful user-confirmed executions,
- unless the workflow qualifies as a pre-verified internal Talent path.

Talents may be revoked or demoted after safety failures.

---

## App Talents vs User-Created Talents

### App Talents

App Talents are:
- pre-coded,
- versioned with the application,
- deterministic,
- and governed by the shipped codebase.

App Talents do not require user-generated manifests.

The application code acts as the authoritative execution definition.

---

### User-Created Talents

User-created Talents are:
- learned,
- taught,
- inferred,
- or converted workflows.

User-created Talents require:
- routing structure,
- manifest validation,
- Delegator approval,
- and verification state.

Only user-created Talents use Talent package structure.

---

## Talent Package Structure

A Talent package represents:
- trusted routes,
- validated steps,
- approved variations,
- and routing definitions.

Unproven Methods are not stored permanently inside Talent packages.

Methods remain in:
- task history,
- or task-chain execution history,
until promoted.

---

## Step Structure

Talent execution is broken into reusable step units.

Example:

```text
01-load-issue-set.md
02-map-code-evidence.md
02a-map-code-evidence-gitnexus.md
03-compare-planning-spec.md
04-report-gaps.md
```

Step numbering defines:
- execution order,
- routing structure,
- and variation inheritance.

---

## Variation Rules

A variation may replace exactly one trusted step.

Example:

Trusted Route:

```text
01 → 02 → 03 → 04
```

Variation Route:

```text
01 → 02a → 03 → 04
```

The variation:
- inherits unchanged trusted steps,
- but the modified route remains Method-state until proven.

If more than one step changes:
- the workflow becomes a new Method candidate,
- not another variation branch.

This prevents uncontrolled branch composition and trust drift.

---

## Trusted Step Inheritance

Trust propagates only through unchanged validated paths.

Unmodified steps may inherit trusted status.

Modified steps:
- lose trusted status,
- require Method validation,
- and must complete the normal promotion cycle.

---

## Routing Files

Talent routing definitions are trusted execution structures.

Routing files define:
- allowed execution paths,
- valid step transitions,
- variation routes,
- and scenario routing behavior.

Routing files are part of the execution trust boundary.

Changes to routing invalidate verification state.

Alice may propose new routes, but:
- users must approve them,
- and Delegator validation must succeed before activation.

---

## Skill Conversion Rules

Skills are not executable EVO units.

Skills are external source material only.

Alice may:
- analyze a skill,
- extract deliverables,
- decompose steps,
- and reconstruct the workflow as:
  - a Method,
  - a Task Chain,
  - a Talent variation,
  - or an existing Talent route.

EVO preserves:
- the intended deliverable,
not necessarily:
- the original execution procedure.

---

## Delegator Relationship

Delegator validates:
- steps,
- tool scopes,
- route structure,
- transitions,
- permissions,
- and execution boundaries.

Delegator does not trust raw markdown directly.

Markdown exists for:
- Alice comprehension,
- user inspection,
- and workflow transparency.

Execution authority belongs to validated routing and manifest structures.

---

## Observation and Teaching Rules

Alice does not passively observe workflows for automation generation.

Method creation requires:
- explicit observation requests,
- explicit conversational teaching,
- or explicit task-analysis approval.

Observation requires intent.

Inference requires approval.

Execution requires Delegator validation.

Promotion requires proof.

---

## Promotion Rules

A Method becomes eligible for Talent promotion after:
- 3 successful user-confirmed executions in a row.

A failed run:
- resets the validation streak.

Safety failures may:
- immediately revoke or demote Talent status.

Operational failures:
- may pause or review a Talent without immediate revocation.

---

## UX Model

User-facing Talents and Methods should appear as:
- tasks,
- subtasks,
- and workflow structures inside the task manager.

Users should interact with:
- readable workflow steps,
- success/failure reporting,
- variation approval,
- and execution history.

Internal routing identifiers may remain technical, but the UI must remain human-readable and operationally understandable.

## Related
- [[EVOconnect — System Map]]
- [[EVO Architecture Bible]]
- [[EVOconnect — Action Bar & Mini Action Bar System.md]]
- [[EVOconnect — Coach Pane Pack Contract.md]]
- [[EVOconnect — Connect Library & Unified Access Layer.md]]
- [[EVOconnect — Hive Node Architecture.md]]
- [[EVOconnect — Method Reconstruction Model.md]]
- [[EVOconnect — Mobile Operational Continuity.md]]
^[source-materials/mirrors/doctrine/EVOconnect — Lightweight Talent Structure Addendum.md]
