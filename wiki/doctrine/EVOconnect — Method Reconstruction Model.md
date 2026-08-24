---
title: Purpose
type: concept
tags: [connect, evo, method, model]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# Purpose

Define how Alice converts observed execution traces into structured, reusable Methods.

This model ensures that Alice learns from her own reconstructed workflows rather than directly from external agent behavior.

# Core Principle

Alice does not learn from what external agents did.

Alice learns from how she would do it.

# Reconstruction Flow

Execution Trace → Decomposition → Method Proposal → Validation → Learning

---

# Step 1 — Execution Trace

Input comes from observable surfaces:

- EVOterminal
- EVObrowser
- MCP/API traces

Trace includes:

- commands
- file changes
- diffs
- outputs
- errors
- retries

---

# Step 2 — Decomposition

Alice breaks the trace into logical steps.

Each step must be:

- explicit
- ordered
- understandable
- tool-agnostic where possible

Bad decomposition:

"Run Codex fix"

Good decomposition:

- identify failing test
- locate relevant file
- modify function
- rerun tests

---

# Step 3 — Method Proposal

Alice converts steps into a Method.

A Method must:

- be reproducible
- be tool-surface compatible
- avoid unnecessary steps
- avoid copying raw trace

Rule:

A Method must not be a direct copy of an execution trace.

It must be a normalized workflow.

---

# Step 4 — Validation

Before learning, the Method must be validated.

Validation requires:

- successful execution
- no contradiction in results
- user approval (when required)

Optional validation:

- repeat execution
- variation testing

---

# Step 5 — Learning

After validation, Alice may:

- store Method
- update Wiki
- influence adapter bias

No direct learning occurs from the original trace.

---

# Method Quality Rules

A valid Method must be:

- clear
- minimal
- deterministic where possible
- safe to execute
- scoped to a domain

---

# Failure Handling

If a Method fails:

- do not promote to learning
- refine or discard
- optionally re-observe execution

---

# Relationship to External Agents

External agents:

- generate execution
- provide suggestions

Alice:

- interprets execution
- reconstructs workflow
- owns the Method

---

# Relationship to Talents

Repeated successful Methods may become Talents.

Criteria:

- repeatable
- high success rate
- user-approved automation

---

# Relationship to Adapters

Adapters are influenced by:

- successful Method patterns
- repeated workflow structures

Adapters are NOT trained on:

- raw execution traces
- external agent outputs

---

# Relationship to the Wiki

The Wiki stores:

- validated Methods
- distilled workflow patterns

The Wiki does not store:

- raw traces as canonical knowledge

---

# Final Principle

Execution is observed.

Methods are reconstructed.

Learning happens from Methods, not from external behavior.

## Related
- [[EVOconnect — System Map]]
- [[EVO Architecture Bible]]
- [[EVOconnect — Action Bar & Mini Action Bar System.md]]
- [[EVOconnect — Coach Pane Pack Contract.md]]
- [[EVOconnect — Connect Library & Unified Access Layer.md]]
- [[EVOconnect — Hive Node Architecture.md]]
- [[EVOconnect — Lightweight Talent Structure Addendum.md]]
- [[EVOconnect — Mobile Operational Continuity.md]]
^[wiki-native — no upstream source]
