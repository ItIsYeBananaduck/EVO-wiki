---
title: "EVOC-103 — OpenClaw Governance Conflicts (Delegator + Bounded Execution)"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-08-19
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-103 — OpenClaw Governance Conflicts (Delegator + Bounded Execution)

> Status: Audit complete (analysis only; no implementation changes).
> Scope: Identify governance conflicts between OpenClaw behavior and EVOconnect/Connect governance model.

## Audit baseline

This conflict inventory is derived from:

- OpenClaw architecture/tooling findings captured in `docs/evofy/EVOC-106-openclaw-architecture-audit.md`.
- Canonical EVOconnect vocabulary defined in `docs/evofy/EVOC-104-openclaw-concept-mapping.md`.
- Delegator governance expectations currently represented in `packages/core/lib/src/delegator/delegator.dart` and `packages/core/lib/src/delegator/safety_rules.dart`.

## Conflict matrix

| #   | Governance area                        | Observed OpenClaw behavior                                                                                                                           | Conflict with Connect governance                                                                                                                    | Required fix direction                                                                                                                                                                         |
| --- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Unrestricted tool execution risk       | Tools are available through runtime policy/sandbox paths, but not held behind a first-class task actionability gate.                                 | Connect requires Delegator-first execution where tools are inaccessible until authorization path is satisfied.                                      | Introduce a **Task Actionability Gate** in the integration boundary: deny all tool calls until Method approval or Talent authorization is resolved; enforce via Delegator as single authority. |
| 2   | Missing approval step contract         | OpenClaw has policy and hook cancellation, but no unified approval contract equivalent to Delegator callback lifecycle.                              | Connect requires explicit approval workflow, fail-safe deny-by-default when approval handler is absent, and deterministic approve/deny transitions. | Route all high-risk/unknown actions through Delegator approval callbacks, with fail-closed behavior and standardized approval event payloads.                                                  |
| 3   | Opaque automation paths                | Plugin hook pathways (especially legacy hook-only compatibility behavior) can execute logic outside explicit Delegator checks.                       | Connect disallows silent execution and requires transparent governance checkpoints before action execution.                                         | Block legacy hook-only plugin path at boundary; require capability-tier contracts that must invoke Delegator policy + approval checks before execution.                                        |
| 4   | Lack of task-scoped audit logs         | OpenClaw session transcripts capture conversation/run history but are not structured per-task audit records with required minimum governance fields. | Connect requires per-task audit entries including task identity, authorization path, execution metadata, and outcome/artifacts.                     | Add structured task audit log schema and emission pipeline for every attempted action (allowed/blocked/denied/executed), linked to task lifecycle states.                                      |
| 5   | Uncontrolled data access surface       | Broad plugin SDK and runtime capability surface can permit data access patterns that are not uniformly classified by governance tier.                | Connect requires data access controls to be explicit, bounded, and policy-versioned under Delegator/tool interfaces.                                | Reduce plugin/tool surface into explicit contract tiers; require declared data scopes, policy version tagging, and deny-by-default for undeclared data channels.                               |
| 6   | Non-versioned governance enforcement   | Reused routing/execution mechanics may not emit Delegator contract version in metadata.                                                              | Connect requires auditability and revalidation whenever governance contracts change.                                                                | Require Delegator contract version in execution metadata and audit logs; enforce revalidation workflow when contract version changes.                                                          |
| 7   | Elevated execution carve-out ambiguity | Elevated execution capabilities exist conceptually, but governance visibility and explicit approval coupling are not guaranteed in all paths.        | Connect requires bounded execution with explicit escalation records and user approval traceability for privileged actions.                          | Gate elevated execution behind explicit escalation action types, mandatory approval, and immutable audit trail entries that include escalation reason and approver outcome.                    |

## Detailed conflict notes

### 1) Unrestricted tool execution (tool hostage rule missing)

OpenClaw has strong policy/sandbox controls, but this is not equivalent to Connect’s “tools unavailable until task is actionable” rule. In Connect, task authorization state must be a hard prerequisite, not an optional policy overlay.

**Required fix direction:** Introduce a hard precondition at the tool dispatcher boundary: `task_actionable == true` before tool invocation. This check must be non-bypassable and delegated to Delegator.

### 2) Missing approval contract parity

Connect Delegator behavior requires explicit approval handling with deny-by-default fallback. OpenClaw lacks a guaranteed equivalent contract shape across all execution paths.

**Required fix direction:** Normalize all approval-required actions into a single approval protocol (request, decision, timeout/absence => deny) and ensure every execution path participates.

### 3) Opaque automation and legacy plugin bypass

Legacy compatibility hooks can allow side effects without explicit Delegator checkpointing, creating a silent automation risk.

**Required fix direction:** Enforce plugin execution only through approved capability contracts and remove/block direct hook-only execution from governed runtime paths.

### 4) Audit logging not task-governance complete

Session logs are useful observability artifacts, but they do not satisfy Connect’s requirement for structured governance records on a per-task basis.

**Required fix direction:** Add task-governance audit schema with minimum required fields:

- task id/title
- action id/type
- authorization path (method/talent/manual)
- approval state + actor + timestamp
- tool call log summary
- execution outcome and output artifact references

### 5) Data-access control model is too broad

Broad capability/plugin surfaces make least-privilege difficult to prove and audit.

**Required fix direction:** Introduce explicit data-scope declarations and deny undeclared access. Every data access path should be classifiable (local files, credentials, remote APIs, personal profile domains, etc.) and policy-enforced.

### 6) Governance-version traceability gap

Without governance contract version metadata, downstream audit and replay/revalidation workflows cannot prove which rule set authorized a run.

**Required fix direction:** Add `delegator_contract_version` and `policy_version` to run/action metadata and force revalidation on version drift.

### 7) Bounded execution + elevation semantics not uniformly explicit

Privileged execution must be both bounded and explainable. Any elevated path without explicit escalation semantics undermines governance guarantees.

**Required fix direction:** Introduce explicit escalation primitives (reason, scope, duration, approval result) and block privileged execution unless escalation is approved and logged.

## Acceptance criteria mapping

- **All governance conflicts documented:** ✅ (7 conflicts across execution, approvals, automation, logging, and data governance).
- **Each conflict includes required fix direction:** ✅ (every item above includes mandatory direction, not implementation).

## Out-of-scope confirmation

- No runtime fixes implemented.
- No refactor performed.
- This document is an audit artifact for follow-on implementation tickets.