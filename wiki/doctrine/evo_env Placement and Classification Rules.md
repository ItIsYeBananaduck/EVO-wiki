---
title: evo_env Placement and Classification Rules
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/evo_env Placement and Classification Rules.md"]
updated: 2026-07-24
---

# evo_env Placement and Classification Rules
## Purpose

Define what belongs inside `.evo_env` versus the normal repo and establish practical classification rules so the protected enclave remains narrow, intentional, and usable.

This exists to solve one specific problem:

If `.evo_env` is used too broadly, the system becomes blind and unworkable.

If it is used too narrowly, privileged material leaks into ordinary repo workflows.

---

## Core Principle

`.evo_env` should be surgical, not total.

It is for what must not be ordinarily inspectable by Alice.

It is not for everything that matters.

---

## Why This Exists

A lot of code is important.

Very little code needs structural blindness.

Without classification rules, teams may:

- overuse `.evo_env`
- hide too much runtime
- break normal repo workflows
- confuse protected runtime artifacts with ordinary code
- weaken the usefulness of Alice in normal engineering work

This note defines a practical boundary.

---

## Classification Model

There are four broad classes of content.

### Class 1 — Normal Repo Content

Ordinary code, documentation, tests, and standard project files.

Alice may:

- read
- search
- summarize
- reason over
- propose changes through governed workflows

This should be the majority of the repo.

### Class 2 — Sensitive Repo Content

Important material that may require tighter governance but does not require structural blindness.

Examples:

- security-relevant app code
- operational configuration
- connector logic
- elevated execution paths

Alice may often still read it, but changes may require more approval.

This usually stays outside `.evo_env`.

### Class 3 — Protected Runtime Artifacts

Artifacts that shape Alice or define privileged runtime behavior but should not be ordinary repo-readable knowledge.

Examples:

- soul files
- protected identity definitions
- privileged shaping configs

These belong in `.evo_env` or equivalent protected storage.

### Class 4 — Blind Protected Workspaces

Work areas that Alice must not inspect directly, often for external-agent-only work or privileged self-referential tasks.

Examples:

- external-agent patch enclave
- protected refactor workspace
- blind review bundles

These belong in `.evo_env`.

---

## What Belongs in `.evo_env`

Good candidates:

- soul source artifacts
- protected runtime identity artifacts
- blindness-enforced privileged configs
- external-agent-only protected work areas
- self-reference sensitive system material
- human-reviewed protected patch bundles
- code or artifacts Alice must never inspect directly

---

## What Does Not Belong in `.evo_env`

Bad candidates:

- the entire repo
- the full runtime
- ordinary application logic
- normal adapters
- most docs
- standard feature code
- anything that needs normal repo readability for Alice to do useful work

If a file only matters, that is not enough reason to hide it.

It should go into `.evo_env` only if ordinary Alice visibility is structurally unsafe.

---

## Repo Rule

The repo should remain mostly normal.

Recommended shape:

repo/

`.evo_env` should be a contained enclave, not the dominant repo structure.

---

## Runtime Rule

Do not place the entire runtime in `.evo_env`.

Why:

- Alice would lose useful visibility into ordinary system behavior
- debugging becomes much harder
- normal engineering workflows degrade
- protected blindness becomes too broad to be practical

Only the small subset that must not be ordinarily inspectable belongs there.

---

## Loader Rule

If something in `.evo_env` must influence runtime, it should do so through:

- protected loader
- compiler
- controlled runtime injection

Not through ordinary module imports visible to Alice as normal repo code.

---

## External Agent Rule

Blind protected workspaces may allow scoped external-agent access.

That access must remain:

- path-scoped
- task-scoped
- reviewable
- non-sovereign

Alice may coordinate these workflows without seeing inside the enclave.

---

## Default Decision Rule

When deciding where something belongs, ask:

### Question 1

Does Alice need ordinary visibility into this to reason effectively about normal system behavior?

If yes:

- keep it outside `.evo_env`

### Question 2

Would ordinary Alice visibility create self-reference, identity, governance, or privileged-boundary risk?

If yes:

- consider `.evo_env`

### Question 3

Is this a runtime-consumed artifact rather than ordinary code?

If yes:

- it likely belongs in protected runtime artifact handling

### Question 4

Is this an external-agent-only or human-review-only workspace?

If yes:

- it likely belongs in blind protected workspace handling

---

## Anti-Patterns

Do not:

- put the entire repo in `.evo_env`
- hide ordinary application code by default
- use `.evo_env` as a vague “important stuff” folder
- treat `.evo_env` as just another code module area
- allow normal repo search/indexing to leak enclave content
- let Alice inspect protected artifacts just because runtime consumes them

---

## Relationship to Other Notes

This note is the operational checklist for:

- [EVO Blind Zone / .evo_env Protected Workspace Model](https://app.notion.com/p/342c72bad01381a4ad10c0da5891b0fc)
- [Protected Runtime Artifacts / Soul File Injection Model](https://app.notion.com/p/342c72bad01381ce97ddd58276a1f7fe)
- Protected System Zones and Privileged Change Policy

Those notes define the concepts.

This note defines the practical placement rules.

---

## Principle

Use `.evo_env` only where structural blindness is required.

Do not turn the protected enclave into the whole system.

---

## End State

A mature EVO repo should have:

- mostly normal readable code
- a narrow protected enclave
- clear classification of privileged artifacts
- runtime consumption without ordinary visibility
- safe external-agent blind workspaces where needed

---

## Summary

`.evo_env` is not for everything important.

It is for what must not be ordinarily visible to Alice.

The clean rule is:

Normal code stays normal.

Protected artifacts go in `.evo_env`.

Blind workspaces go in `.evo_env`.

The whole repo does not.

---

## Related Notes

- [EVO Blind Zone / .evo_env Protected Workspace Model](https://app.notion.com/p/342c72bad01381a4ad10c0da5891b0fc)
- [Protected Runtime Artifacts / Soul File Injection Model](https://app.notion.com/p/342c72bad01381ce97ddd58276a1f7fe)
- Protected System Zones and Privileged Change Policy
- [Delegator — Execution Governance Doctrine](https://app.notion.com/p/342c72bad01381e088ecc512452813e4)
^[{src_rel}]
