---
title: "EVOC-107 — Phased EVOfication Refactor Roadmap"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-08-19
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-107 — Phased EVOfication Refactor Roadmap

> Status: Planning artifact (no implementation).
> Current phase: Phase 2 — Minimal clone (Phase 1 artifacts complete: EVOC-103, EVOC-104, EVOC-106).
> Objective: Convert OpenClaw audit findings into an executable, dependency-aware refactor roadmap for EVOconnect.

## Inputs used

This roadmap is based on prior EVOfication analysis artifacts:

- `docs/evofy/EVOC-106-openclaw-architecture-audit.md` (architecture inventory and subsystem disposition)
- `docs/evofy/EVOC-104-openclaw-concept-mapping.md` (canonical terminology mapping)
- `docs/evofy/EVOC-103-openclaw-governance-conflicts.md` (Delegator and bounded execution conflicts)
- `docs/evofy/EVOC-102-openclaw-reuse-wrap-replace-delete-matrix.md` (subsystem-level reuse/wrap/refactor/replace/delete decisions)
- `docs/evofy/EVOC-101-minimum-viable-openclaw-clone-boundary.md` (minimum clone boundary for governed bootstrap)

These artifacts (`EVOC-101`, `EVOC-102`, `EVOC-103`, `EVOC-104`, `EVOC-106`) are treated as the prerequisite set of audit and framing work.

## Phase plan

### Phase 1 — Audit and mapping

**Goal:** Freeze source-of-truth understanding for architecture, vocabulary, and governance gaps.

**Scope**

- Consolidate OpenClaw subsystem inventory and disposition (reusable/wrapper/refactor/replace/delete).
- Lock canonical terminology mapping for EVOconnect language.
- Lock governance conflict inventory with required fix directions.

**Deliverables**

- Architecture audit baseline document.
- Concept/vocabulary mapping matrix.
- Governance conflict matrix.

**Entry criteria**

- EVOfication initiative is approved for planning.

**Exit criteria**

- Audit, mapping, and governance conflict docs are approved by architecture + platform owners.
- No unresolved contradictions between architecture disposition and governance constraints.

---

## Phase 2 — Minimal clone

**Goal:** Produce a minimal, non-production EVOfication clone boundary that can execute controlled “hello path” runs.

**Scope**

- Create isolated integration boundary from OpenClaw runtime shell into EVOconnect workspace.
- Preserve baseline control-plane behavior required for deterministic local runs.
- Stub/disable non-essential legacy behavior not needed for the first execution slice.

**Deliverables**

- Minimal clone package/module layout.
- Bootstrapping notes and deterministic local run path.
- Risk log for clone divergences from source runtime.

**Entry criteria**

- Phase 1 exit criteria met.

**Exit criteria**

- Clone boots in local/dev environment.
- One deterministic end-to-end “session → run → transcript/audit event” path verified.
- Legacy compatibility path usage is explicitly identified and tracked.

---

## Phase 3 — Rename and strip legacy behavior

**Goal:** Remove semantic and structural ambiguity by applying EVOconnect naming and deleting unsafe legacy pathways.

**Scope**

- Apply canonical renames from EVOC-104 across the cloned boundary.
- Strip or hard-block legacy hook-only behavior that can bypass governance.
- Reduce broad plugin/runtime surfaces to explicitly named contract edges.

**Deliverables**

- Rename map implementation plan + migration notes.
- Legacy behavior removal list with disposition per path (deleted/blocked/wrapped).
- Contract boundary index for remaining extension points.

**Entry criteria**

- Phase 2 minimal clone is stable.

**Exit criteria**

- Canonical vocabulary is reflected in architecture/docs/interfaces for the clone layer.
- Legacy hook-only compatibility path cannot execute inside the governed runtime path.
- Remaining extension points are enumerated and ownership assigned.

---

## Phase 4 — Add governance (Delegator)

**Goal:** Make Delegator the non-bypassable authority for actionability, approval, and bounded tool execution.

**Scope**

- Introduce task actionability gate (deny-by-default before tool execution).
- Wire approval lifecycle contract (request/approve/deny/timeout→deny).
- Add required governance metadata/versioning (`delegator_contract_version`, `policy_version`).
- Define per-task structured audit record requirements.

**Deliverables**

- Delegator integration blueprint.
- Governance event schema + audit schema.
- Escalation model for elevated execution with required audit fields.

**Entry criteria**

- Phase 3 legacy stripping complete for governed execution paths.

**Exit criteria**

- No tool path executes without Delegator gate enforcement.
- Approval-required actions fail closed when approval handler is absent.
- Governance metadata and task audit minimum fields are defined and validated in test plans.

---

## Phase 5 — Plugin system alignment

**Goal:** Align plugin architecture to EVOconnect contract tiers while preserving useful capabilities.

**Scope**

- Reclassify plugin capabilities into contract tiers.
- Enforce declared data scopes and deny undeclared access.
- Standardize plugin lifecycle hooks that are governance-aware.

**Deliverables**

- Plugin contract tier specification.
- Capability/data-scope declaration schema.
- Compatibility policy (supported/deprecated/blocked plugin behaviors).

**Entry criteria**

- Phase 4 governance controls are specified and integrated at boundaries.

**Exit criteria**

- Plugin execution paths are compatible with Delegator and bounded tools.
- Legacy compatibility behaviors are blocked or migrated to approved tiers.
- Data-access paths are classifiable and policy-addressable.

---

## Phase 6 — UI integration

**Goal:** Integrate governed runtime behavior into EVOconnect user/operator surfaces.

**Scope**

- Connect runtime and governance states into UX surfaces (task status, approvals, denied actions, audit visibility).
- Ensure terminology and task ownership language follow EVOC-104 mapping.
- Expose actionable error states for governance denials and escalation flows.

**Deliverables**

- UI integration spec for task/governance state surfaces.
- Event-to-UI mapping for approvals, denials, and audit outcomes.
- UX copy map aligned to canonical terms.

**Entry criteria**

- Phase 5 plugin alignment complete for core execution paths.

**Exit criteria**

- Users/operators can see and act on governance checkpoints in UI flows.
- Task and action outcomes are visible with policy-aware explanations.
- No UI path suggests ungoverned/autonomous execution semantics.

---

## Phase 7 — Hive/Swarm alignment

**Goal:** Complete distributed alignment by mapping node/presence semantics to Hive and defining Swarm integration strategy.

**Scope**

- Wrap node identity/presence into Hive-compatible models.
- Add Lease Holder semantics (designation, transfer, offline fallback).
- Define Swarm replacement strategy (no direct OpenClaw subsystem transplant).

**Deliverables**

- Hive alignment spec with lease semantics.
- Swarm architecture decision package.
- Risk and rollout plan for distributed execution.

**Entry criteria**

- Phase 6 UX integration ready for distributed-state presentation.

**Exit criteria**

- Hive wrapper contracts and lease rules are defined and testable.
- Swarm path is explicitly selected (replace strategy) with implementation backlog.
- Distributed governance invariants are documented.

## Dependency graph

```text
Phase 1 (Audit + mapping)
  ↓
Phase 2 (Minimal clone)
  ↓
Phase 3 (Rename + strip legacy)
  ↓
Phase 4 (Delegator governance)
  ↓
Phase 5 (Plugin system alignment)
  ↓
Phase 6 (UI integration)
  ↓
Phase 7 (Hive/Swarm alignment)
```

### Cross-phase dependency notes

- Phase 4 depends on Phase 3 removing/blocking bypass paths; otherwise Delegator guarantees are not enforceable.
- Phase 5 depends on Phase 4 governance primitives to classify plugin behavior safely.
- Phase 6 depends on Phases 4–5 so UI can represent true governance states.
- Phase 7 depends on governance + UI semantics being stable before distributed orchestration alignment.

## First executable milestone

### Milestone M1: “Governed hello-path runtime”

**Target phase boundary:** End of Phase 4.

**Definition**
A minimal EVOfication runtime slice that can:

1. create/select a task context,
2. evaluate actionability through Delegator,
3. request approval when required,
4. execute (or deny) a bounded tool action,
5. emit structured per-task audit records with contract/version metadata.

**Why this is first executable milestone**

- It proves the highest-risk invariant early: **no silent or bypassed execution**.
- It validates both runtime viability (from Phase 2) and governance correctness (Phase 4).
- It creates a safe foundation for plugin and UI expansion without reworking core safety assumptions.

**Milestone acceptance checklist**

- Deterministic hello-path run in dev succeeds.
- Deny-by-default behavior is observable when approval is missing.
- Audit record includes task identity, authorization path, decision, and outcome metadata.
- At least one negative-path test case (blocked/denied action) is demonstrated.

## Out of scope confirmation

- No implementation tasks executed in this issue.
- No code/runtime refactor delivered here.
- This artifact is planning input for follow-on execution tickets.