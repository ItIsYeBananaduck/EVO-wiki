---
title: Purpose
type: concept
tags: [connect, evo, model]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# Purpose

Define how EVOconnect determines what is considered truth when interacting with tools, external agents, and execution environments.

This model establishes that Alice learns from observed execution, not from agent claims or intentions.

# Core Principle

Do not trust declared tool use.

Trust observed execution traces.

# Truth Hierarchy

## Observed Execution (Source of Truth)

The only reliable source of truth is what actually executed.

Examples:

- terminal commands that ran
- files that were created or modified
- diffs applied to the codebase
- API calls that completed
- browser actions performed
- test results and outputs
- errors and retries

This data is considered ground truth.

## Structured Results (Validated Output)

Results returned from tools are valid when:

- execution is confirmed
- outputs are complete
- no contradiction exists

Examples:

- test results
- API responses
- command output

## Agent Declarations (Untrusted)

External agents may declare:

- what they plan to do
- what they believe they did
- what tools they claim to have used

These are not considered reliable.

Example:

“I ran the tests” is not proof that tests were run.

# Observable Surfaces

All execution must pass through controlled surfaces that provide traceability.

## EVOterminal

Provides:

- command history
- process execution
- file system changes
- stdout / stderr
- exit codes

This is the primary surface for development and system-level work.

## EVObrowser

Provides:

- navigation history
- DOM interaction
- user interface actions
- extracted content

Used when structured APIs are unavailable.

## MCP / API Surfaces

When using MCP servers or APIs, observability must include:

- request payload
- response data
- success/failure state
- side effects, if applicable

# External Agent Rule

External agents such as ChatGPT, Codex, Claude, and similar systems are:

- advisors
- planners
- execution participants

They are not sources of truth.

# Learning Rule

Alice must learn from:

- observed execution traces
- validated outcomes
- user-approved results

Alice must not learn from:

- agent claims
- unverified outputs
- assumed tool usage

# Execution vs Narrative

Narrative = what the agent says happened.

Execution = what actually happened.

Only execution is used for:

- learning
- validation
- Method reconstruction
- Talent promotion

# Failure Handling

If a mismatch exists between agent claim and observed execution, observed execution overrides agent claim.

# Observability Requirement

All Tool Surfaces must produce:

- traceable logs
- auditable steps
- reproducible execution

If a tool cannot produce observable traces, it must not be used for learning.

# Relationship to Method Reconstruction

Observable traces are the input to Method creation.

Flow:

Execution trace → Alice decomposition → Method proposal → Validation → Learning

# Relationship to Adapters

Adapters must never be trained on:

- agent claims
- unverified outputs

Adapters may be influenced by:

- validated execution patterns
- repeated successful workflows

# Relationship to the Wiki

The Wiki stores:

- distilled patterns from validated execution
- structured knowledge derived from traces

The Wiki must not store:

- raw agent narratives as truth
- unverified claims

# Final Principle

Alice does not learn from what agents say.

Alice learns from what actually happens.

## Related
- [[EVOconnect — System Map]]
- [[EVO Architecture Bible]]
- [[EVOconnect — Action Bar & Mini Action Bar System.md]]
- [[EVOconnect — Coach Pane Pack Contract.md]]
- [[EVOconnect — Connect Library & Unified Access Layer.md]]
- [[EVOconnect — Hive Node Architecture.md]]
- [[EVOconnect — Lightweight Talent Structure Addendum.md]]
- [[EVOconnect — Method Reconstruction Model.md]]
^[wiki-native — no upstream source]
