---
title: "EVOC-211 — Method Proposal Interaction Contract"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-04-01
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-211 — Method Proposal Interaction Contract

> Status: Draft contract (architecture only; no UI implementation).
> Date: 2026-04-01.
> Parent: EVOC-210 — Connect Core Interaction Contract.

## 1) Purpose

Define the user-facing interaction contract for **Method proposals** in Connect so proposal behavior is consistent across chat and future surfaces.

This contract covers:

- what a user must see,
- what actions a user can take,
- what system states/events are produced,
- and what is explicitly disallowed.

## 2) Core invariants

1. A Method proposal is a **review artifact**, not an automatic mutation.
2. The user always has enough context to decide (goal, plan, expected outcome, optional preview).
3. User intent is captured only via the approved actions in this contract.
4. **No direct Method mutation** may occur from proposal rendering or user questions.
5. Execution/actionability remains governed by Delegator and downstream approval contracts.

## 3) Required proposal payload (what user sees)

Every Method proposal shown to a user MUST include:

1. **Goal**
   - A concise statement of what the Method is intended to achieve.
2. **Plan (subtasks)**
   - Ordered subtasks that summarize the proposed execution path.
3. **Expected outcome**
   - Observable result if the proposal is approved and later executed.
4. **Optional preview**
   - Additional detail (examples, timeline estimate, or sample output) when available.

If any required field (`goal`, `plan`, `expectedOutcome`) is missing, the proposal is non-actionable and must be marked incomplete.

## 3.5) Optional proposal metadata

In addition to the required payload fields, a Method proposal MAY include the following optional metadata:

### replaces

- **Type**: `string` (proposalId or UUID)
- **Cardinality**: Optional, single value
- **Semantics**: When present, indicates that this proposal supersedes a prior proposal identified by the referenced `proposalId`. The system MUST automatically transition the referenced proposal's state to `superseded` when the new proposal is presented, **but ONLY if the referenced proposal is in a terminal state** (`approved`, `denied`, or `superseded`). If the referenced proposal is in a non-terminal state (e.g., `pending_review`), the system MUST reject the new proposal with an error indicating that the prior proposal must reach a terminal state before it can be replaced.
- **Validation rules**:
  - MUST reference a valid, existing `proposalId`
  - MUST NOT reference itself (no circular superseding)
  - **MUST reference a proposal in a terminal state** (`approved`, `denied`, or `superseded`)
  - If the referenced proposal is in `pending_review` or another non-terminal state, the system MUST reject the new proposal
- **Example (valid flow)**:

  ```json
  // Step 1: Original proposal reaches terminal state
  {
    "proposalId": "prop_old_456",
    "goal": "Initial approach",
    "state": "approved"  // Terminal state
  }

  // Step 2: New proposal replaces the approved one
  {
    "proposalId": "prop_new_123",
    "replaces": "prop_old_456",
    "goal": "Updated approach to achieve the same objective",
    "plan": ["revised subtask 1", "revised subtask 2"],
    "expectedOutcome": "Improved outcome with better performance"
  }
  // Result: prop_old_456 transitions to "superseded"
  ```

- **Example (invalid flow - rejected)**:

  ```json
  // Attempt to replace a non-terminal proposal
  {
    "proposalId": "prop_pending_789",
    "state": "pending_review"  // Non-terminal state
  }

  // This will be REJECTED:
  {
    "proposalId": "prop_new_999",
    "replaces": "prop_pending_789",  // ERROR: references non-terminal proposal
    "goal": "Attempted replacement"
  }
  // System response: Reject with error "Cannot replace proposal in pending_review state"
  ```

When a proposal with a `replaces` field is presented, the system behavior is:

1. Validate that the referenced `proposalId` exists
2. **Validate that the referenced proposal is in a terminal state** (`approved`, `denied`, or `superseded`)
3. If validation fails (non-terminal state), **REJECT the new proposal** and return an error
4. If validation succeeds, transition the referenced proposal to `superseded` state
5. Emit a `method_proposal.superseded` event for the old proposal
6. Emit a `method_proposal.presented` event for the new proposal
7. Link the two proposals in the audit trail for traceability

This ensures consistent implementation of superseding behavior across all producers and consumers and prevents ambiguity when multiple proposals for the same method are in flight.

## 4) Allowed user actions

A user may perform exactly these proposal interactions:

1. **Approve**
2. **Deny**
3. **Ask questions**

No additional action class is part of this version of the contract.

## 5) Interaction semantics

### 5.1 Approve

When a user approves:

- proposal state transitions to `approved`,
- an immutable approval event is recorded (`actor`, `timestamp`, `proposalId`),
- proposal becomes eligible for downstream execution flow,
- and any actual Method/task mutation occurs only in downstream governed execution stages (not in this interaction layer).

### 5.2 Deny

When a user denies:

- proposal state transitions to `denied`,
- a denial event is recorded (`actor`, `timestamp`, optional `reason`),
- proposal is no longer actionable unless re-proposed as a new revision,
- no Method mutation occurs.

### 5.3 Ask questions

When a user asks questions:

- proposal state remains `pending_review`,
- question/answer exchanges are appended as proposal discussion context,
- proposal identity is preserved (same `proposalId`) unless a revised proposal is explicitly emitted,
- no Method mutation occurs.

## 6) State model (proposal-level)

### 6.1 States

- `pending_review`
- `approved`
- `denied`
- `superseded` (optional, when replaced by a revised proposal)

### 6.2 Transition rules

- `pending_review -> approved` via Approve only.
- `pending_review -> denied` via Deny only.
- `pending_review -> pending_review` via Ask questions.
- **Terminality**: `approved`, `denied`, and `superseded` are terminal states. Once a proposal reaches a terminal state, it cannot transition to any other state except `superseded` (when replaced by a newer proposal).
- **Reversibility**: An `approved` proposal CANNOT be revoked to `denied` on the same proposal instance. If a denial is required after approval, a new proposal must be created to supersede it.
- **Superseded trigger**: Creating a new `proposalId` that references a prior proposal via a "replaces" field automatically marks the prior proposal as `superseded`. **This transition ONLY applies to proposals already in terminal states** (`approved`, `denied`, or `superseded`). If a new proposal attempts to replace a proposal in a non-terminal state (e.g., `pending_review`), the system MUST reject the new proposal. This prevents ambiguity when multiple proposals are in flight.
- No direct transition from `denied -> approved` on the same proposal instance.

## 7) Event contract

Each interaction must emit a typed event at proposal scope:

- `method_proposal.presented`
- `method_proposal.approved`
- `method_proposal.denied`
- `method_proposal.question_asked`
- `method_proposal.answered`
- `method_proposal.superseded` (optional, if revision flow is used)

Minimum event fields:

- `proposalId`
- `methodRef` (or equivalent method identifier)
- `userId` (actor)
- `timestamp`
- `stateBefore`
- `stateAfter`
- `surface` (chat/web/mobile/etc.)

**Note**: These seven items are the minimum required fields. Events MAY include additional action-specific fields to capture relevant context. For example:

- `method_proposal.denied` events MAY include a `"reason"` field (see Section 5.2).
- `method_proposal.question_asked` events MAY include a `"questionContent"` field to record the user's question.

## 8) Non-mutation guardrails (critical)

The following are explicitly prohibited in this contract stage:

- mutating Method definition directly from proposal rendering,
- mutating Method definition directly from question handling,
- implicit auto-approval,
- implicit deny on timeout without explicit policy contract.

Any mutation request must be routed through formal execution/update contracts outside EVOC-211.

## 9) Acceptance criteria

This issue is complete when:

1. The proposal payload contract requires `goal`, `plan`, and `expectedOutcome` with optional `preview`.
2. The only user actions are `approve`, `deny`, and `ask questions`.
3. State transition rules are explicit and enforce no direct mutation.
4. Event emissions are defined for all allowed interactions.
5. Contract explicitly states: **No direct mutation of Methods**.

## 10) Out of scope

- visual styling and component design,
- platform-specific rendering differences,
- animation/micro-interactions,
- advanced multi-step negotiation UX beyond ask/answer loop.