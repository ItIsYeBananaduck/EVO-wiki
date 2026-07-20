---
type: audit-finding
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-108 — OpenClaw Execution Model vs EVOconnect Task Model

> Status: Audit complete (analysis only; no implementation or refactor).
> Last reviewed: 2026-03-25.

## 1) Scope

This document compares how OpenClaw executes actions/runs against EVOconnect’s task-based model, with focus on:

- action trigger paths,
- execution state handling,
- retry behavior,
- outcome tracking,
- My Tasks vs Alice Tasks alignment,
- Method-based and Talent-based execution,
- approval flow compatibility,
- logging and audit requirements.

Dependencies referenced in issue context:

- EVOC-49 (task execution state machine stabilization)
- EVOC-50 (Alice chat execution wiring)
- EVOS1-8 (shared task runtime extraction)

## 2) Baseline evidence

### OpenClaw-side evidence (already captured in repository audit docs)

- `docs/evofy/EVOC-106-openclaw-architecture-audit.md`
- `docs/evofy/EVOC-103-openclaw-governance-conflicts.md`
- OpenClaw concept docs referenced inside EVOC-106 (`architecture`, `agent-loop`, `multi-agent`, plugin/gateway docs).

### EVOconnect-side evidence

- Canonical terminology split (`My Tasks` / `Alice Tasks`): `docs/evofy/EVOC-104-openclaw-concept-mapping.md`
- Runtime and execution outcomes: `packages/core/lib/src/runtime/task_runtime.dart`
- Governance and approval contract: `packages/core/lib/src/delegator/delegator.dart`
- Stateful execution model, transitions, retry/escalation behavior: `flutter_app/lib/features/alice/domain/task_execution_state_machine.dart`
- Coverage of approval and retry/escalation paths: `flutter_app/test/task_execution_state_machine_test.dart`

## 3) OpenClaw execution model (audit summary)

From the prior OpenClaw audit baseline:

1. **Action trigger model**
   - Primarily message-driven agent runs, plus cron/automation-triggered runs.
   - Trigger unit is a session/run request rather than a first-class durable task entity.

2. **Execution state handling**
   - Gateway-owned session and run lifecycle, with lane-queued execution.
   - State is represented operationally through run progression and transcript/events.

3. **Retry handling**
   - Retries/re-attempts are loop/routing behaviors, not a strongly typed task-state transition contract.
   - No explicit equivalent of task retry API with guardrails (e.g., only failed → reviewing).

4. **Outcome tracking**
   - Durable history exists as session transcript + run events.
   - Not equivalent to structured per-task governance/audit records with mandatory fields.

## 4) EVOconnect task execution model (current repo baseline)

1. **Action trigger model**
   - Execution is task-centric (`ConnectTask`) through runtime + delegator contracts.
   - Trigger enters `TaskRuntime.executeTask(...)` with explicit `task`, `action`, and executor closure.

2. **Execution state handling**
   - Explicit task statuses and transitions (e.g., `newTask`, `reviewing`, `needsApproval`, `ready`, `working`, terminal states).
   - Transition guards prevent invalid jumps and enforce approval/retry entry points.

3. **Retry handling**
   - Retry is explicit (`retryExecution`) and constrained (failed → reviewing via dedicated path).
   - Failure/retry signals update escalation metadata (`failureCount`, repeated/persistent error triggers, conflicting tool triggers).

4. **Outcome tracking**
   - Task runtime emits typed outcomes (`completed`, `blocked`, `denied`, `failed`, `skippedAwaitingApproval`).
   - Task logs + runtime events + escalation details provide structured execution/accountability data.

5. **Approval and governance contract**
   - Delegator validates safety rules before execution.
   - Approval-required actions use callback flow.
   - Missing approval handler is fail-safe deny-by-default.

## 5) Side-by-side mapping

| Dimension                | OpenClaw execution model                                        | EVOconnect task model                                                  | Compatibility assessment                                                   |
| ------------------------ | --------------------------------------------------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Trigger primitive        | Session message / run trigger                                   | First-class task + action execution request                            | **Mismatch** (OpenClaw run unit must map into ConnectTask/action envelope) |
| Execution state          | Implicit/operational run progression + transcripts              | Explicit finite state machine + guarded transitions                    | **Mismatch** (requires explicit state projection)                          |
| Retry semantics          | Loop-level behavior, less typed at task boundary                | Dedicated retry APIs and transition restrictions                       | **Mismatch** (needs deterministic retry contract translation)              |
| Outcome primitive        | Transcript/run artifacts                                        | Structured runtime outcomes + task logs                                | **Partial** (raw signal reusable; schema not equivalent)                   |
| Approval model           | Policy/hooks/sandbox; no guaranteed universal approval contract | Delegator-mediated approval, fail-closed when handler missing          | **Mismatch** (must route all approval-required paths through Delegator)    |
| Audit model              | Session/run history                                             | Per-task structured governance records                                 | **Mismatch** (new audit projection required)                               |
| Ownership lens           | Generic tasks/runs                                              | Explicit UI/ownership split: My Tasks vs Alice Tasks                   | **Mismatch** (requires ownership attribution rules)                        |
| Method/Talent expression | Plugin/capability/routing concepts                              | Explicit execution path metadata (`method`, `talent`, delegation path) | **Partial** (conceptual overlap, needs canonical mapping and contracts)    |

## 6) Required transformation rules (for OpenClaw → Connect runtime alignment)

These rules define what must exist before execution parity can be claimed.

### Rule group A — Trigger and identity normalization

1. Every OpenClaw run trigger must produce or attach to a deterministic `ConnectTask` ID.
2. The integration layer must classify trigger source (`message`, `cron`, automation) and persist source metadata.
3. Each trigger must set an initial ownership lens: `My Tasks` (user-owned) or `Alice Tasks` (assistant-executed), with reversible mapping only through explicit delegation events.

### Rule group B — State projection and transition enforcement

1. OpenClaw run lifecycle signals must be projected onto EVO task states.
2. Invalid state jumps must be rejected by the task state machine (no direct bypass to terminal/working states).
3. `needsApproval -> working` may only occur through explicit `approve` resolution event.
4. `failed -> reviewing` may only occur through explicit retry flow.

### Rule group C — Method/Talent execution path normalization

1. Every executable action must carry a normalized execution-path record:
   - `method` reference when method-based,
   - `talent` reference when talent-based,
   - delegation metadata when routed across actors/agents.
2. If execution path is ambiguous, task is non-actionable and must remain blocked until path resolution.

### Rule group D — Approval and actionability gate

1. Tool/action execution must be blocked unless task is actionable under Delegator policy.
2. Actions marked approval-required must invoke Delegator approval callbacks.
3. If approval callback is unavailable, action is denied by default and logged as governance denial.
4. Deny/approve decisions must produce explicit audit events with actor + timestamp + reason.

### Rule group E — Retry and escalation normalization

1. Failed execution increments retry metadata at task scope.
2. Repeated failures and persistent-error signatures must raise escalation-required status.
3. Conflicting tool recommendations must be captured as conflict signals and escalate deterministically.
4. Retry attempts must preserve prior failure/audit lineage (no overwrite semantics).

### Rule group F — Outcome and audit projection

1. Every action attempt must emit standardized outcome category:
   - completed / blocked / denied / failed / skipped-awaiting-approval.
2. Task-scoped audit entries must include minimum fields:
   - task ID, task title, timestamps,
   - ownership lens (My Tasks/Alice Tasks),
   - authorization path (method/talent/manual),
   - approval state + decision actor,
   - tool/action summary,
   - outcome category,
   - output artifact references.
3. Session transcript content may be retained as supplemental evidence, not as the sole audit record.

## 7) Gap and incompatibility inventory

1. **No first-class task parity in OpenClaw runtime primitive**
   - Connect requires task-first semantics; OpenClaw is run/session-first.
2. **Approval contract non-equivalence**
   - Connect requires guaranteed deny-by-default callback contract.
3. **State-machine strictness mismatch**
   - Connect relies on explicit transition invariants absent from OpenClaw baseline.
4. **Audit shape mismatch**
   - Transcript-style history does not satisfy Connect governance-grade task audit requirements.
5. **Ownership/modeling mismatch**
   - OpenClaw generic task language does not encode My Tasks/Alice Tasks split required by Connect UX and governance tracing.

## 8) Acceptance criteria check (EVOC-108)

- **Clear mapping between OpenClaw execution and Connect task model:** ✅
  - Sections 3–5 provide normalized model comparison and dimensional mapping.
- **Identified gaps and incompatibilities:** ✅
  - Section 7 enumerates structural incompatibilities.
- **Required transformation rules defined:** ✅
  - Section 6 defines explicit rule groups A–F for runtime alignment.

## 9) Out-of-scope confirmation

- No implementation changes made.
- No runtime refactor performed.
- This is an analysis artifact for downstream execution/runtime tickets.