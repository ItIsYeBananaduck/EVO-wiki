---
type: audit-finding
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-104 — OpenClaw Concept Mapping to EVOconnect

> Status: Canonical vocabulary mapping for OpenClaw → EVOconnect terminology alignment.
> Scope: Terminology mapping only (no code renames, no implementation changes).

## Purpose

This document defines the canonical vocabulary used when translating OpenClaw concepts into EVOconnect architecture and product language.

## Canonical mapping matrix

| OpenClaw term      | EVOconnect canonical concept | Classification | Notes                                                                                            |
| ------------------ | ---------------------------- | -------------- | ------------------------------------------------------------------------------------------------ |
| agents             | Alice runtime                | rename         | “agent” references are normalized to runtime-backed Alice execution contexts.                    |
| tools              | bounded tools                | rename         | Preserve tool semantics, but always framed as governed/bounded capabilities.                     |
| workflows          | Methods                      | rename         | Workflow intent is retained, but canonical planning/execution unit is Method.                    |
| automation         | governed delegation          | split          | “Automation” is too broad; split into scheduling/triggering + Delegator-governed execution.      |
| extensions/plugins | plugin contract              | split          | Legacy plugin vocabulary maps to explicit contract tiers and compatibility policy.               |
| tasks              | My Tasks / Alice Tasks       | split          | User-facing task views split by ownership and execution lens while sharing task lifecycle model. |

## Legacy vocabulary disposition

All legacy terms are classified with one of: **keep**, **rename**, **split**, **remove**.

| Legacy term                           | Disposition | Canonical replacement / policy                                                           |
| ------------------------------------- | ----------- | ---------------------------------------------------------------------------------------- |
| agents                                | rename      | Use **Alice runtime**.                                                                   |
| tools                                 | rename      | Use **bounded tools**.                                                                   |
| workflows                             | rename      | Use **Methods**.                                                                         |
| automation                            | split       | Use **governed delegation** + explicit trigger/schedule language when needed.            |
| extensions                            | split       | Use **plugin contract** terminology.                                                     |
| plugins                               | split       | Use **plugin contract** terminology.                                                     |
| tasks                                 | split       | Use **My Tasks** (user-owned surface) and **Alice Tasks** (assistant execution surface). |
| autonomous mode (ungoverned phrasing) | remove      | Replace with governed delegation language that implies explicit Delegator policy gates.  |
| unrestricted tool execution phrasing  | remove      | Replace with bounded-tool and approval-gated phrasing.                                   |

## Usage guidance

1. Use EVOconnect canonical terms in new docs/specs by default.
2. If legacy OpenClaw wording must be retained for migration context, include the canonical equivalent in the same section.
3. Avoid introducing new synonyms for Method, bounded tools, or governed delegation.
4. Prefer “plugin contract” over broad “plugin system” phrasing when discussing extension boundaries.
5. In user-visible task UX copy, prefer **My Tasks** and **Alice Tasks** over generic “tasks” when ownership/execution context matters.

## Dependency alignment

This mapping is informed by:

- EVOC-49 (task execution state machine stabilization)
- EVOC-50 (Alice chat/task flow wiring)

The mapping can proceed independently, but terminology should remain consistent with those initiatives.

## Out of scope confirmation

- No implementation changes.
- No codebase-wide rename/refactor.
- No runtime extraction work (tracked separately).