---
title: Purpose
type: concept
tags: [connect, evo, execution, model]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# Purpose

Define how external capabilities are exposed to Alice and how execution is governed within EVOconnect.

This model establishes the foundation for tool usage, execution visibility, fallback behavior, and Delegator enforcement.

# Core Principle

All external capabilities are treated as Tool Surfaces.

Tool Surfaces provide capabilities, not authority.

Alice does not execute tools directly.

All execution is mediated through Delegator.

# Tool Surface Types

## MCP Servers (Primary)

MCP servers are the preferred integration model.

They provide:

- Tools (actions)
- Resources (readable data)

Examples:

- filesystem
- GitHub
- databases
- calendars
- messaging systems

MCP servers standardize interaction and reduce ambiguity.

## APIs (Structured Fallback)

APIs are used when MCP is not available.

These include:

- custom backend services
- legacy integrations
- third-party endpoints

APIs should be wrapped into MCP-compatible interfaces when possible.

## OAuth Integrations (Access Layer)

OAuth integrations provide:

- identity
- authentication
- authorization

They do not provide direct execution control.

Examples:

- ChatGPT
- Codex
- Google
- GitHub

OAuth enables access but does not bypass Delegator.

## EVOterminal (Controlled Execution Surface)

EVOterminal is a governed execution environment.

It provides:

- command execution
- file system interaction
- process visibility
- full output tracing

Used for:

- development workflows
- local/system operations
- external agent execution when observability is required

## EVObrowser (Controlled Web Surface)

EVObrowser is a governed web interaction layer.

It provides:

- page navigation
- DOM interaction
- scoped visibility
- controlled automation

Used when structured APIs are not available.

# Tool Preference Order

Alice should always prefer the highest-structure interface available:

1. MCP servers (standardized, structured)
2. APIs (direct structured access)
3. OAuth integrations (authorized environments)
4. EVOterminal / EVObrowser (controlled fallback execution)

# Fallback Rule

If no structured interface exists, Alice may use EVOterminal or EVObrowser.

All fallback execution must:

- pass through Delegator
- be observable
- be auditable
- produce execution traces

# Critical Constraint

Fallback surfaces must not replace structured integrations when they are available.

Alice must not default to browser or terminal usage when MCP or API options exist.

# Execution Flow

All tool usage follows this flow:

User → Alice → Delegator → Tool Surface → Result → Alice

- Alice proposes actions
- Delegator validates and approves
- Tool Surface executes
- Results are returned and observed

# Governance Rules

- Alice must not directly execute tools
- All tool calls must pass through Delegator
- Execution must be observable
- All actions must be auditable
- Tool access must be scoped and permissioned

# Observability Requirement

Tool execution must produce observable traces.

These include:

- commands executed
- files accessed or modified
- API calls
- browser actions
- outputs and errors

Observable traces are required for:

- validation
- debugging
- learning
- Method reconstruction

# Final Principle

Tool Surfaces provide access.

Delegator enforces control.

Execution must remain visible.

Alice operates through governed, observable, and auditable actions.

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
