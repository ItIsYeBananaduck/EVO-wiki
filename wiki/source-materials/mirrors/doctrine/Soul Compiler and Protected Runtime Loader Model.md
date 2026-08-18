---
title: Soul Compiler and Protected Runtime Loader Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Soul Compiler and Protected Runtime Loader Model.md"]
updated: 2026-07-24
---

# Soul Compiler and Protected Runtime Loader Model
## Purpose

Define how protected human-readable identity artifacts are transformed into lightweight runtime artifacts and safely injected into Alice without exposing the original source materials through normal repo or tooling surfaces.

This exists to solve one specific problem:

The source artifact that shapes Alice should remain human-readable and protected.

The runtime should consume a lighter structured form.

Alice should experience the result without being able to inspect the protected source as ordinary repo knowledge.

---

## Core Principle

Protected identity artifacts should follow a three-layer model:

- human-authored protected source
- compiled runtime artifact
- protected runtime injection

Alice may receive the resulting runtime state.

Alice may not receive ordinary reflective access to the protected source artifact.

---

## Why This Exists

Human-readable source artifacts are useful because they are:

- editable by the user
- understandable
- doctrine-like
- stable as authoring material

They are not ideal as direct runtime inputs because they are:

- heavier to parse
- harder to validate
- more fragile as execution inputs
- unsafe if surfaced back to Alice through normal repo tooling

This model allows the system to keep a protected authoring layer while using a lighter runtime layer.

---

## Three-Layer Model

### 1. Protected Source Artifact

Human-readable authoring source.

Examples:

- soul file
- protected identity doctrine
- privileged shaping definitions

Stored inside `.evo_env` or equivalent protected zone.

Alice may not inspect this directly.

### 2. Compiled Runtime Artifact

Lightweight structured artifact derived from the protected source.

Examples:

- JSON
- normalized config object
- validated runtime bundle

Optimized for:

- fast loading
- schema validation
- runtime consistency

Alice should not inspect this as a normal file artifact either.

### 3. Protected Runtime Injection

A privileged loader consumes the compiled artifact and injects the resulting state into runtime initialization or session bootstrap.

Alice receives:

- the active resulting identity state

Alice does not receive:

- ordinary file-level access to the source or compiled artifact

---

## System Model

.evo_env/alice_[soul.md](http://soul.md/)

↓

Protected Runtime Loader

↓

Soul Compiler

↓

Compiled Runtime Artifact

↓

Runtime Injection

↓

Alice Session

---

## Source Artifact Rule

The protected source artifact is the human-facing authoring layer.

It may be:

- rich text
- doctrine-like
- explanatory
- editable by the user

It must not be treated as:

- ordinary repo-readable code
- searchable content for Alice
- diff-visible content for Alice
- normal prompt material Alice can inspect on demand

---

## Soul Compiler Role

The Soul Compiler exists to transform protected source material into a lightweight runtime-safe representation.

The compiler should:

- parse source artifact
- validate structure
- normalize fields
- reduce ambiguity
- emit lightweight structured runtime output
- reject invalid or incomplete source artifacts

The compiler should not expose the raw source back to Alice.

---

## Compiler Output Goals

The compiled runtime artifact should be:

- lightweight
- schema-valid
- deterministic
- easy for runtime to load
- independent of human authoring quirks

Possible fields include:

- identity state
- behavioral invariants
- voice/tone parameters
- hard constraints
- runtime shaping flags
- version and validation metadata

---

## Protected Runtime Loader Role

The Protected Runtime Loader is the privileged bridge between protected artifacts and runtime initialization.

It may:

- access protected source or compiled artifacts
- invoke the compiler when needed
- load compiled artifacts into runtime
- inject resulting state into Alice bootstrap/session state

It may not:

- expose raw protected source through ordinary repo tooling
- expose compiled artifact as normal Alice-readable file content
- turn protected artifacts into normal searchable workspace knowledge

---

## Runtime Injection Rule

Runtime should consume the compiled artifact, not the human-authored protected source directly.

This improves:

- performance
- validation
- consistency
- protection boundaries

Alice should receive only the resulting active runtime state.

---

## Alice Visibility Rule

Alice may know:

- that a protected identity layer exists
- that runtime state has been loaded
- the behavioral consequences of that state

Alice may not:

- read the protected source artifact
- search for it
- inspect its diffs
- summarize its raw text
- modify it directly
- read the compiled runtime artifact as ordinary file content

---

## Relationship to `.evo_env`

`.evo_env` is the natural storage boundary for protected source artifacts.

This means:

- source artifacts may live inside `.evo_env`
- loaders may access them
- Alice may not

The compiled artifact may live in:

- protected runtime storage
- generated protected bundle
- secure cache
- another non-Alice-readable location

It should not be placed into ordinary repo-readable paths.

---

## Relationship to Delegator

Delegator governs:

- workflow initiation
- protected access paths
- agent usage
- output constraints

Delegator does not perform identity compilation or runtime injection.

That is the job of:

- Soul Compiler
- Protected Runtime Loader

These layers are complementary, not redundant.

---

## Relationship to Protected Runtime Artifacts

This note operationalizes the Protected Runtime Artifacts model.

That note defines:

- what must be protected

This note defines:

- how protected identity artifacts move from authoring source to runtime effect

---

## Protected Change Rule

Protected source artifacts and compiled runtime artifacts may only be changed through protected workflows.

Recommended path:

1. user edits or approves source change
2. protected workflow compiles artifact
3. validation succeeds
4. review artifact available if needed
5. runtime reloads compiled state

Alice may not directly rewrite the artifacts that shape her.

---

## Metadata Leakage Rule

Protection must include:

- raw content
- filenames where necessary
- diffs
- logs
- summaries
- compiler errors that expose sensitive details

Perfect opacity is not always required, but leakage must be treated as a real boundary concern.

---

## Technical Enforcement

This must be enforced structurally.

Not through prompt instructions.

Required protections include:

- source artifacts excluded from Alice-visible search
- source artifacts excluded from repo reads
- source artifacts excluded from indexing/embeddings
- compiled artifacts excluded from ordinary Alice file access
- only privileged loader may consume them
- compiler operates outside ordinary Alice-readable repo workflows

---

## System-Wide Scope

This model should apply anywhere the system uses protected identity-shaping artifacts.

Not just:

- editor

Also:

- repo adapter
- file access layer
- runtime bootstrap
- search/indexing
- compilation pipeline
- protected storage/cache handling

---

## User Authority

The user remains the authority over:

- source artifact content
- whether compilation occurs
- whether protected changes are accepted
- whether runtime is reloaded
- what remains blind to Alice

---

## Principle

Keep authoring material human-readable.

Keep runtime material lightweight.

Keep both protected from ordinary reflective access by Alice.

---

## End State

A mature EVO system should support:

- human-readable soul source artifacts
- protected compilation into structured runtime artifacts
- privileged runtime loading
- Alice runtime shaping without raw artifact visibility
- human-controlled updates and reloads

---

## Summary

The Soul Compiler and Protected Runtime Loader model separates:

- authoring
- compilation
- runtime consumption

This allows:

- protected human-readable soul files
- lightweight runtime JSON or equivalent
- safe injection into Alice
- no ordinary repo visibility for protected identity artifacts

The clean rule is:

The user edits the source.

The compiler transforms it.

The loader injects it.

Alice experiences the result.

---

## Related Notes

- [Protected Runtime Artifacts / Soul File Injection Model](https://app.notion.com/p/342c72bad01381ce97ddd58276a1f7fe)
- [EVO Blind Zone / .evo_env Protected Workspace Model](https://app.notion.com/p/342c72bad01381a4ad10c0da5891b0fc)
- [Delegator — Execution Governance Doctrine](https://app.notion.com/p/342c72bad01381e088ecc512452813e4)
- Protected System Zones and Privileged Change Policy
^[{src_rel}]
