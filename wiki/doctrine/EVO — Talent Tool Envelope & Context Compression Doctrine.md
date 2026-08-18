---
title: EVO — Talent Tool Envelope & Context Compression Doctrine
type: concept
tags: [context, doctrine, evo, talent]
sources:
  - source-materials/mirrors/doctrine/EVO — Talent Tool Envelope & Context Compression Doctrine.md
updated: 2026-07-23
---
# EVO — Talent Tool Envelope & Context Compression Doctrine

## Purpose

Define how Alice manages:

- MCP server activation
- tool exposure
- delegation environments
- context lifecycle
- memory pressure
- resumable execution

within EVOconnect and the broader EVOsystem.

This doctrine exists to ensure:

- lightweight execution
- deterministic tool access
- reduced context waste
- reduced RAM pressure
- reduced token usage
- safer delegation
- resumable long-running workflows
- scalable multi-device execution

---

## Core Principle

Installed capability is not active capability.

Alice may have access to many tools, MCP servers, and execution systems, but only the minimum required subset may be active during execution of a Talent.

Tool activation must be:

- scoped
- temporary
- task-bound
- observable
- revocable

---

## Tool Envelope Doctrine

### Definition

A Tool Envelope is the bounded execution environment attached to a Talent.

It defines:

- which tools may be used
- which MCP servers may be active
- which capabilities are forbidden
- context budget constraints
- approval requirements
- execution limits

### Purpose

Tool Envelopes exist to:

- reduce context pollution
- reduce startup overhead
- reduce token consumption
- reduce RAM pressure
- reduce accidental tool misuse
- improve execution determinism
- improve auditability
- improve security boundaries

---

## Installed vs Active Capability

### Installed Capability

Installed capability represents all tools, MCP servers, and integrations available on the device.

Examples:

- GitHub MCP
- Linear MCP
- Supabase MCP
- Context7
- GitNexus
- Playwright
- Slack
- Browser automation
- File system access

Installed capability is passive.

It does not imply active availability during execution.

### Active Capability

Active capability represents the subset of tools temporarily activated for the current Talent.

Only active capability is exposed to:

- Alice
- delegated specialists
- execution chains
- automation systems

---

## Talent Tool Envelope

Every Talent must define the following.

### Required Tools

Tools necessary for execution.

Example:

- GitHub MCP
- GitNexus

### Optional Tools

Tools Alice may activate if needed.

Example:

- Context7
- Browser automation

### Forbidden Tools

Tools explicitly disallowed during execution.

Example:

- Supabase write access
- terminal execution
- filesystem mutation

### Context Budget

Maximum allowed execution context.

Examples:

- token budget
- RAM budget
- retrieval budget
- history carry-forward limit

### Execution Budget

Operational limits.

Examples:

- maximum retries
- maximum delegated sessions
- maximum parallel agents
- maximum runtime duration

---

## Delegated Specialist Envelopes

### Principle

Delegated specialists must not inherit full system capability by default.

Before delegation:

1. Alice evaluates the task.
2. Alice selects the required Tool Envelope.
3. Alice activates only the required MCP servers.
4. Alice disables unnecessary capabilities.
5. Delegation begins.

### Purpose

This ensures:

- lightweight delegated sessions
- reduced context usage
- safer external execution
- lower token overhead
- reduced hallucinated tool usage
- improved determinism

---

## Dynamic MCP Activation

### Rule

MCP servers must be activated dynamically per Talent whenever possible.

They should not remain globally active unless required for:

- core runtime stability
- persistent sync infrastructure
- user-approved background services

### Examples

#### PR Review Talent

Active:

- GitHub
- GitNexus

Inactive:

- Supabase
- Browser
- Slack
- Playwright

#### Calendar Scheduling Talent

Active:

- Calendar
- Notification system

Inactive:

- GitHub
- GitNexus
- Browser automation

#### Supabase Migration Talent

Active:

- Supabase
- GitHub
- GitNexus

Inactive:

- Slack
- Browser
- Calendar

---

## Context Compression Doctrine

### Principle

Long-running execution must not rely on indefinitely preserved context windows.

Talents must be resumable from durable task state.

---

## Compression Lifecycle

### Phase 1 — Execute

Alice executes bounded Talent steps using the active Tool Envelope.

### Phase 2 — Record

Alice records:

- completed steps
- unresolved blockers
- outputs
- artifacts
- state transitions
- retrieval anchors
- semantic references

### Phase 3 — Compress

Alice generates a compressed continuation summary containing:

- current objective
- completed work
- pending work
- execution decisions
- known blockers
- relevant references
- retrieval anchors

Compression must preserve operational continuity while minimizing token footprint.

### Phase 4 — Clear

Alice may:

- terminate delegated sessions
- clear active context
- release KV cache
- deactivate MCP servers
- unload unused Tool Envelopes

### Phase 5 — Resume

To resume execution:

1. Alice restores the Talent state.
2. Alice reloads the required Tool Envelope.
3. Alice retrieves compressed context.
4. Alice performs semantic retrieval for missing details.
5. Alice continues execution.

---

## Durable Task State Doctrine

### Principle

Execution continuity must come from structured state, not raw conversation history.

Talents must preserve:

- step completion state
- execution order
- dependency state
- artifacts
- outputs
- semantic anchors
- unresolved blockers

This state becomes the authoritative execution memory.

---

## Semantic Retrieval Rule

### Principle

Compressed execution may omit low-priority details.

Missing detail retrieval must occur through:

- semantic search
- graph traversal
- linked notes
- execution artifacts
- structured logs
- task chain references

not through unbounded historical replay.

---

## RAM-Aware Execution

### Principle

Alice must scale execution behavior to device capability.

Low-memory devices must favor:

- shorter sessions
- aggressive compression
- smaller Tool Envelopes
- sequential execution
- lightweight retrieval
- reduced MCP activation

High-memory devices may permit:

- larger active context
- broader envelopes
- parallel execution
- longer-running sessions

---

## Delegator Integration

Delegator governs Tool Envelope enforcement.

Delegator may:

- activate tools
- deactivate tools
- reject forbidden tool usage
- terminate envelope violations
- enforce context budgets
- enforce execution budgets

Alice may not bypass Delegator enforcement.

---

## Audit Requirements

All Tool Envelope transitions must be auditable.

Audit logs should include:

- activated MCP servers
- deactivated MCP servers
- delegated specialist environment
- context compression events
- context restoration events
- envelope violations
- forced shutdowns
- retry counts

---

## Summary

EVO uses:

- dynamic capability activation
- Talent-bound Tool Envelopes
- resumable execution
- context compression
- semantic retrieval
- RAM-aware orchestration

to ensure:

- scalable local execution
- low-overhead delegation
- deterministic workflows
- safer automation
- lightweight mobile operation
- efficient multi-device cognition

Execution continuity comes from:

- structured task state
- semantic retrieval
- compressed operational memory

not from permanently preserved context windows.

## Related
- [[EVO Architecture Bible]]
- [[EVO Wiki — One Alice, Many Rooms.md]]
- [[EVO — Adapter Training System.md]]
- [[EVO — Cognition Layer.md]]
- [[EVO — Context Layer.md]]
- [[EVO — Cross-App Context Continuity.md]]
- [[EVO — Global Adapter Distribution Model.md]]
- [[EVO — Pane Pack Architecture.md]]
^[source-materials/mirrors/doctrine/EVO — Talent Tool Envelope & Context Compression Doctrine.md]
