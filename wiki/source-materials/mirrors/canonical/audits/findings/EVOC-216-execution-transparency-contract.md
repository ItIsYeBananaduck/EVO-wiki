---
title: "EVOC-216 — Execution Transparency Contract"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-04-02
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-216 — Execution Transparency Contract

> Status: Draft contract (architecture only; no UI implementation).
> Date: 2026-04-02.
> Parent: EVOC-210 — Connect Core Interaction Contract.

## 1) Purpose

Define the cross-surface **execution transparency contract** so a user can always tell:

1. what is happening,
2. which task is active,
3. and which step is currently running.

This contract standardizes lifecycle communication across:

- chat,
- task view,
- and notifications.

It also establishes a shared lifecycle language the system must communicate:

- `reviewing`,
- `awaiting_approval`,
- `executing`,
- `completed`.

## 2) Core invariants

1. Execution transparency is mandatory at all times while work is in-flight.
2. Every surfaced execution state must answer three user questions:
   - **what is happening now**,
   - **which task is active now**,
   - **which step is running now** (or why step detail is unavailable).
3. Lifecycle wording and meaning must be semantically consistent across chat, task view, and notifications.
4. Surfaces may differ in density, but not in state truth.
5. If a surface cannot render full detail, it must provide a resolvable pointer to richer detail (for example, link/open action to task view).
6. The system must never imply completion while state is still `reviewing`, `awaiting_approval`, or `executing`.

## 3) Transparency lifecycle model

### 3.1 Canonical states

The execution transparency lifecycle uses these canonical states:

- `reviewing`
- `awaiting_approval`
- `executing`
- `completed`

### 3.2 State semantics

1. **`reviewing`**
   - System is analyzing intent, constraints, and plan readiness.
   - Execution has not started.
2. **`awaiting_approval`**
   - A proposal/decision is ready and waiting for explicit user action.
   - Execution remains blocked pending user response.
3. **`executing`**
   - System is actively running approved work.
   - Active task and current step context must be surfaced.
4. **`completed`**
   - Execution reached a terminal successful outcome for the execution scope.
   - Terminal summary must be available.

### 3.3 Allowed transitions

- `reviewing -> awaiting_approval`
- `awaiting_approval -> executing` (after approval)
- `executing -> completed`

Optional loops (allowed when applicable):

- `reviewing -> reviewing` (continued analysis/refinement)
- `awaiting_approval -> awaiting_approval` (follow-up question/clarification without decision)
- `executing -> executing` (task/step progress updates)

Disallowed:

- direct `reviewing -> completed`
- direct `awaiting_approval -> completed` without explicit resolved outcome path

## 4) Required transparency payload

Every surfaced transparency update MUST include these minimum fields:

- `executionId`
- `transparencyState` (`reviewing` | `awaiting_approval` | `executing` | `completed`)
- `headline` (human-readable "what is happening" summary)
- `activeTaskId` (nullable only when no task is active)
- `activeTaskLabel` (nullable only when no task is active)
- `activeStepId` (nullable when step detail is unavailable/not applicable)
- `activeStepLabel` (nullable when step detail is unavailable/not applicable)
- `stepDetailStatus` (`available` | `unavailable` | `not_applicable`)
- `timestamp`
- `surface`

Rules:

1. If `transparencyState = executing`, `activeTaskId` and `activeTaskLabel` must be non-null.
2. If `stepDetailStatus = available`, both `activeStepId` and `activeStepLabel` must be non-null.
3. If step detail is unavailable, the payload must explicitly set `stepDetailStatus = unavailable` and must not fabricate step fields.
4. `headline` must be state-congruent (for example, an executing headline must not claim completion).

## 5) Cross-surface rendering contract

### 5.1 Chat

Chat must show:

1. current transparency state label,
2. concise "what is happening" summary,
3. active task indicator when executing,
4. active step indicator or explicit step-unavailable status,
5. transitions as chronological updates (not silent replacement only).

### 5.2 Task view

Task view must show:

1. full state badge/lifecycle marker,
2. active task and neighboring task context,
3. current step details for active task when available,
4. last update timestamp,
5. terminal completion summary when completed.

### 5.3 Notifications

Notifications must show (at minimum):

1. execution identity (or resolvable reference),
2. current transparency state,
3. short "what is happening" line,
4. direct action/deep link to fuller detail in chat or task view.

Notification-specific rule:

- Notifications may be abbreviated, but must preserve truthful state and must not collapse `reviewing`, `awaiting_approval`, and `executing` into one ambiguous "in progress" status without state qualifier.

## 6) Event contract

Required event types:

- `execution_transparency.reviewing`
- `execution_transparency.awaiting_approval`
- `execution_transparency.executing`
- `execution_transparency.completed`
- `execution_transparency.active_task_changed`
- `execution_transparency.active_step_changed`
- `execution_transparency.step_detail_unavailable`

Minimum event fields:

- `executionId`
- `eventType`
- `transparencyState`
- `headline`
- `activeTaskId` (nullable)
- `activeStepId` (nullable)
- `stepDetailStatus`
- `timestamp`
- `surface`

Consistency requirements:

1. State transition events must be emitted before or atomically with visible UI state change.
2. `active_task_changed` and `active_step_changed` events are valid only when `transparencyState = executing`.
3. When step data drops out during execution, `step_detail_unavailable` must be emitted with preserved active task context.

## 7) Guardrails and non-goals

Prohibited:

- hiding current transparency state while execution is in-flight,
- showing task/step values without execution correlation (`executionId`),
- rendering stale completion status after re-entry into non-terminal states,
- inferring active step text when telemetry does not provide it,
- surfacing contradictory states across surfaces for the same `executionId` and timestamp window.

Non-goals for this issue:

- visual styling/tokens,
- platform-specific layout details,
- animation choreography,
- transport-specific notification provider mechanics.

## 8) Acceptance criteria

This issue is complete when:

1. Contract explicitly defines the four canonical states: `reviewing`, `awaiting_approval`, `executing`, `completed`.
2. Contract requires user-facing visibility of:
   - what is happening,
   - active task,
   - active step (or explicit unavailability).
3. Cross-surface requirements are documented for chat, task view, and notifications.
4. Required payload fields and event types are defined with step-unavailable semantics.
5. Guardrails prevent ambiguous/contradictory status communication.

## 9) Out of scope

- scoring, prioritization, or scheduling policy,
- approval-policy logic beyond visibility semantics,
- persistence backend implementation choices,
- UX copy tone guidelines beyond state truthfulness.