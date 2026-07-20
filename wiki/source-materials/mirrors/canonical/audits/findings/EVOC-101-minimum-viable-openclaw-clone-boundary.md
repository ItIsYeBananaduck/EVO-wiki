---
type: audit-finding
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-101 — Minimum Viable OpenClaw Clone Boundary

> Status: Boundary defined (analysis only; no implementation changes).
> Scope: Define the smallest OpenClaw subset to clone for EVOconnect bootstrap.

## Objective

Define the **minimum viable clone boundary** from OpenClaw into EVOconnect so runtime bootstrap can begin without importing broad, unsafe, or architecture-conflicting subsystems.

This boundary is intentionally narrow and optimized for:

- governed execution,
- a single coherent runtime path,
- and compatibility with upcoming runtime ownership wiring (EVOS1-29, EVOS1-36).

## Boundary principles

1. **Clone only execution primitives that are governance-compatible.**
2. **Prefer wrappers over direct subsystem transplants** when OpenClaw semantics diverge from EVOconnect contracts.
3. **Reject mode-fragmented behavior** in MVP; use one canonical runtime path.
4. **Keep tools hostage to governance** (Delegator-first; deny-by-default).

## Minimum viable clone boundary (Foundation)

The following OpenClaw-aligned capability areas are in-boundary for MVP cloning/adaptation.

| Capability area                                        | Include in MVP                      | Why it is in-boundary                                                                           | Clone posture                                                                             |
| ------------------------------------------------------ | ----------------------------------- | ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Gateway/session routing primitives                     | Yes                                 | Provides minimal message/session/run routing backbone needed to stand up runtime orchestration. | **Clone + wrap** under EVO RuntimeManager ownership contracts.                            |
| Run dispatch and queue mechanics                       | Yes                                 | Supplies core execution flow skeleton needed for task processing.                               | **Clone + refactor** into explicit Connect task lifecycle handoff points.                 |
| Bounded tool execution hooks (policy/sandbox-adjacent) | Yes (narrow slice)                  | Useful as low-level enforcement plumbing when subordinated to Delegator gates.                  | **Clone selectively**; enforce Delegator authorization precondition before any tool call. |
| Plugin/capability registration skeleton                | Yes (minimal contract surface only) | Needed for extensibility bootstrap, but only through governed capability tiers.                 | **Clone minimally** and narrow interfaces.                                                |
| Session persistence primitives                         | Yes (minimal)                       | Required to retain execution context continuity for MVP runtime behavior.                       | **Clone + adapt** to EVO task/audit model boundaries.                                     |

## Explicit exclusions (Not in MVP boundary)

The following are explicitly out-of-boundary for EVOC-101 MVP clone definition.

| Excluded area                                                          | Exclusion reason                                                                                    |
| ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Unrestricted execution paths                                           | Violates Delegator-first governance and deny-by-default actionability requirements.                 |
| Non-governed systems or hook-only bypass paths                         | Conflicts with mandatory approval and auditable enforcement guarantees.                             |
| Mode-based runtime fragmentation                                       | MVP requires one canonical governed execution path; multiple mode semantics are deferred.           |
| Full OpenClaw plugin ecosystem/marketplace behavior                    | Surface area is too broad for least-privilege bootstrap; introduce only minimal governed contracts. |
| Channel-specific expansion features not required for runtime bootstrap | Not required to prove core runtime boundary and governance scaffold.                                |
| OpenClaw-native task substitutes (session transcript as task record)   | Not equivalent to Connect task lifecycle and structured audit requirements.                         |

## Foundation vs future split

### Foundation (this issue)

- Define and freeze minimum clone boundary.
- Constrain clone scope to routing/dispatch/tooling primitives required for governed runtime bootstrap.
- Explicitly exclude unsafe and architecture-divergent subsystems.

### Future features (deferred beyond EVOC-101)

- Expanded plugin compatibility layers.
- Additional channel integrations beyond bootstrap needs.
- Advanced orchestration capabilities (swarm-style behavior, lease-holder semantics, federated behaviors).
- Rich policy packs and non-essential automation pathways.
- Non-critical developer ergonomics/features that do not affect core governed runtime path.

## Dependency alignment

- **EVOS1-29 (`AliceRuntimeManager`)** and **EVOS1-36 (`packages/ai-runtime` shell)** are partial dependencies for final runtime wiring.
- EVOC-101 can proceed now because boundary definition is architectural and does not require runtime implementation completion.

## Acceptance criteria mapping

- **Minimal clone boundary defined:** ✅
  - Foundation modules listed with rationale and clone posture.
- **Non-essential modules explicitly excluded:** ✅
  - Unrestricted execution, non-governed systems, and mode fragmentation are explicitly excluded.

## Out-of-scope confirmation

- No cloning performed.
- No implementation performed.
- No runtime wiring changes performed.