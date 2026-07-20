---
type: audit-finding
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-214 — Execution Control Contract

> Status: Draft contract (architecture only; no UI implementation).
> Date: 2026-04-01.
> Parent: EVOC-210 — Connect Core Interaction Contract.

## 1) Purpose

Define the user-facing control contract for **in-flight execution** so user control actions are consistent, explicit, and non-mutating across chat and future surfaces.

This contract covers:

- control actions available during execution,
- required semantics for stop and pause,
- override flow via pause → interaction → new Method proposal,
- and strict non-mutation guardrails.

## 2) Core invariants

1. Execution control is explicit and user-initiated.
2. Two primary user interruption actions are valid: **stop** and **pause**. Additionally, **resume** is permitted only as a continuation from the `paused` state.
3. `stop` is a hard cancel of the active execution.
4. `pause` is a temporary halt that preserves resumable context.
5. Override/replan happens only through: **pause → interaction → new Method proposal**.
6. **No hidden mutation** is allowed from any control action.
7. Any change to plan/method must be represented as an auditable proposal revision, never as implicit in-place mutation.

## 3) Control action contract

### 3.1 stop (hard cancel)

When a user invokes `stop`, the system MUST:

1. transition execution from `active` or `paused` to `canceled`,
2. halt further task dispatch for that execution,
3. mark in-flight work as terminated according to downstream runtime policy,
4. emit a terminal cancellation event (see Section 6),
5. reject resume requests on the same `executionId` (see Section 3.4 for invalid action handling).

Rules:

- `stop` is terminal for the execution instance.
- `stop` does not mutate Method definition, task plan structure, or proposal contents.
- Any follow-up execution must start from a new proposal/execution flow.
- Duplicate `stop` requests on the same execution are handled per Section 3.4 (idempotent semantics).

### 3.2 pause (temporary halt)

When a user invokes `pause`, the system MUST:

1. transition execution from `active` to `paused`,
2. suspend further dispatch while preserving current execution context,
3. retain enough state for either resume or override decision,
4. emit a pause event (see Section 6).

Rules:

- `pause` is non-terminal.
- `pause` alone must not alter Method/task definitions.
- While paused, user interaction may gather clarification, constraints, or corrections.
- Duplicate `pause` requests on an already-paused execution are handled per Section 3.4 (idempotent semantics).

### 3.3 resume (control continuation)

When a user invokes `resume`, the system MUST:

1. verify the execution is in `paused` state (reject if not per Section 3.4),
2. transition execution from `paused` to `active`,
3. continue dispatch using the preserved execution context and `executionId`,
4. emit a resume event (see Section 6).

Rules:

- `resume` is valid **only** from `paused` state.
- `resume` is rejected if execution is `active`, `canceled`, `completed`, or `failed` (see Section 3.4 for invalid action handling).
- `resume` continues the same execution context without modification.
- Duplicate `resume` requests on an already-active execution are handled per Section 3.4 (idempotent semantics).

### 3.4 Invalid action handling

When a user invokes a control action that is redundant or invalid for the current execution state (e.g., `pause` when already `paused`, `resume` when already `active`, `stop` when already `canceled`, or any control action after a terminal state), the system MUST:

1. reject the request deterministically without modifying execution state,
2. return a standardized rejection response:
   - HTTP: `409 Conflict` with a structured error body, or
   - RPC: error code `InvalidExecutionState`
   - (Other status codes/error codes are reserved for distinct failure classes, such as authentication failures, validation errors, or server errors),
3. emit a `execution_control.control_action_rejected` event containing:
   - `executionId` — the execution identifier,
   - `attemptedAction` — the action that was attempted (e.g., `pause`, `resume`, `stop`),
   - `currentState` — the current state of the execution (e.g., `paused`, `active`, `canceled`),
   - `rejectionReason` — a human-readable explanation (e.g., "Execution is already paused"),
   - all minimum event fields defined in Section 6.

Idempotency guarantees:

- Duplicate requests for actions that have already completed (e.g., a second `stop` on an already `canceled` execution) MUST be rejected with the same semantics as above.
- Services MUST NOT silently succeed or return inconsistent responses for redundant actions.
- All invalid or redundant actions MUST be auditable via the rejection event stream.

## 4) Override and replan flow

Override is not an in-place mutation. Override MUST follow this sequence:

1. Execution is `paused`.
2. User interacts (questions, corrections, new constraints, intent updates).
3. System emits a **new Method proposal** reflecting changes.
4. User reviews proposal using EVOC-211 proposal interaction contract.
5. If approved, downstream execution starts as a new execution instance.

Required guarantees:

- Original paused execution remains auditable and unchanged.
- Revised behavior is represented by a new `proposalId` (and later a new `executionId`).
- An override MUST create a new `proposalId` and MAY include the `replaces` field (as defined in EVOC-211) to link to the original proposal, but ONLY when the referenced proposal is already in a terminal proposal state (`approved`, `denied`, or `superseded`) per EVOC-211.
- If the referenced proposal is not in a terminal proposal state at override time, the override request MUST be rejected per EVOC-211 validation rules. The system MUST NOT automatically terminalize a non-terminal proposal as part of the override operation.
- If the original proposal is not yet in a terminal state, the new override proposal MUST omit the `replaces` field and instead include explicit lineage metadata (e.g., `replacedBy` or `parentProposalId` and a `reason` field) so traceability is preserved without violating EVOC-211 constraints.

## 5) Execution state model (control scope)

### 5.1 States

- `active`
- `paused`
- `canceled` (terminal)
- `completed` (terminal; defined by execution visibility/outcome contracts)
- `failed` (terminal; defined by execution visibility/outcome contracts)

### 5.2 Transition rules

- `active -> paused` via `pause`.
- `paused -> active` via `resume`.
- `active -> canceled` via `stop`.
- `paused -> canceled` via `stop`.
- No transition from `canceled -> active`.
- No control action may transition directly to a mutated plan state.

## 6) Event contract (execution control)

Each control action MUST emit a typed event.

Required event types:

- `execution_control.pause_requested`
- `execution_control.paused`
- `execution_control.resume_requested`
- `execution_control.resumed`
- `execution_control.stop_requested`
- `execution_control.canceled`
- `execution_control.control_action_rejected` (see Section 3.4)
- `execution_control.override_interaction_started` (optional but recommended)
- `execution_control.override_proposal_emitted` (required when override produces a new proposal)

Minimum event fields:

- `executionId`
- `proposalId` (nullable when not yet linked)
- `userId` (actor)
- `timestamp`
- `stateBefore`
- `stateAfter` — **for `*_requested` events** (e.g., `pause_requested`, `resume_requested`, `stop_requested`), this MUST be set to the same value as `stateBefore` (i.e., no state transition has occurred yet); the actual state transition is reflected in the corresponding completion event (e.g., `paused`, `resumed`, `canceled`). For all other events, this reflects the new state after the action.
- `reason` (nullable; for `*_requested` events, may indicate user intent or context)
- `surface`

For `control_action_rejected` events, include:

- `attemptedAction` — the control action that was attempted (e.g., `pause`, `resume`, `stop`)
- `rejectionReason` — human-readable explanation of why the action was rejected

For override-proposal events, include:

- `newProposalId`
- `replaces` (type: string, nullable but recommended; references the original proposalId as per EVOC-211)

## 7) Guardrails (no hidden mutation)

The following are explicitly prohibited:

- changing task order/content as a side effect of `pause`, `resume`, or `stop`,
- silently rewriting Method definition from runtime interaction,
- auto-generating revised plans without emitting a new proposal artifact,
- continuing execution under modified assumptions without explicit user approval path,
- treating pause-time interaction as implicit approval.

Any requested behavioral change discovered during execution must flow through explicit proposal lifecycle contracts.

## 8) Acceptance criteria

This issue is complete when:

1. Execution control actions are constrained to `stop` (hard cancel) and `pause` (temporary halt), with explicit `resume` semantics from paused state.
2. `stop` terminal behavior and `pause` non-terminal behavior are documented.
3. Override flow is explicitly defined as **pause → interaction → new Method proposal**.
4. Event types and minimum payload fields are defined for control actions.
5. Contract explicitly states: **No hidden mutation**.

## 9) Out of scope

- visual styling or control placement,
- platform-specific UX (chat vs mobile vs web layout),
- scheduler/runtime internals for cancellation mechanics,
- policy decisions on auto-timeout/auto-resume beyond explicit user controls.