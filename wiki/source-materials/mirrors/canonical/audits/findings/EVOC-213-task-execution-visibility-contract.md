---
type: audit-finding
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-213 — Task Execution Visibility Contract

> Status: Draft contract (architecture only; no UI implementation).
> Date: 2026-04-01.
> Parent: EVOC-210 — Connect Core Interaction Contract.

## 1) Purpose

Define the user-facing visibility contract for **task execution** so execution progress is represented consistently across chat and future surfaces.

This contract covers:

- timeline representation (task-level, not step-level),
- how each task is encoded in the timeline,
- focused-task detail behavior,
- and what is explicitly out of scope.

## 2) Core invariants

1. The execution timeline models **tasks**, not internal method steps.
2. Each task in sequence is represented by exactly one timeline **dot**.
3. Dot semantics are stable and shared across all surfaces:
   - position = sequence order,
   - fill = task progress,
   - animation = actively executing task.
4. Internal method steps are visible only in **focused task** context.
5. Timeline state must remain understandable even when step details are unavailable.

## 3) Timeline model (task scope)

### 3.1 Unit of representation

- The timeline unit is a **task**.
- Steps MUST NOT appear as top-level timeline entries.

### 3.2 Task dot contract

For every task in a method execution, the UI contract MUST provide one dot with the following behavior:

1. **Position (sequence)**
   - Dot index maps to task order in the execution plan.
   - Re-ordering is not allowed unless the underlying plan order changes.
2. **Fill (progress)**
   - Encodes current task completion percentage/state.
   - Fill progression must be monotonic per task (no visual rollback unless task is explicitly reset/replanned by upstream contracts).
3. **Animation (active)**
   - Animation indicates that the task is currently executing.
   - At most one task dot may be in active animation at any point in a single execution stream.

## 4) Focused-task detail contract

When a user focuses/selects a specific task dot, the system MUST expose task-internal details:

1. **Method steps**
   - Ordered list of the steps that compose that task.
2. **Current step**
   - Explicit indicator of which step is in progress (if any).
3. **Step progress**
   - Progress state/percentage for the currently focused task's step flow.

If step-level telemetry is unavailable, the focused view must still show task-level state with a clear "step detail unavailable" status instead of inventing step data.

## 5) Visibility states (task-level)

Minimum task-level states required for visibility semantics:

- `queued` (known task, not started),
- `active` (currently executing),
- `completed` (finished successfully),
- `blocked` (cannot proceed due to dependency/approval/error),
- `failed` (terminated unsuccessfully).

Mapping rule:

- Dot animation is only valid for `active`.
- Dot fill reflects progress from `queued` to `completed` and may freeze in `blocked`/`failed`.

## 6) Event contract (visibility updates)

Each task visibility update MUST emit a typed event that can drive all surfaces consistently.

Required event types:

- `task_visibility.task_registered`
- `task_visibility.task_progressed`
- `task_visibility.task_activated`
- `task_visibility.task_focused`
- `task_visibility.task_completed`
- `task_visibility.task_blocked`
- `task_visibility.task_failed`

Minimum fields per event:

- `executionId`
- `taskId`
- `taskIndex`
- `timestamp`
- `stateBefore`
- `stateAfter`
- `progress` (task-level)
- `surface`

For focused-task events (`task_visibility.task_focused`), include:

- `stepIndex` (nullable)
- `stepProgress` (nullable)
- `stepCount` (nullable)

## 7) Guardrails and non-goals

The following are explicitly prohibited in this contract stage:

- representing method steps as timeline-level dots,
- using dot animation to indicate anything except active task execution,
- implying execution completion from focus/selection interactions,
- using styling-only differences as a substitute for state semantics.

## 8) Acceptance criteria

This issue is complete when:

1. Timeline contract explicitly states: **timeline = tasks, not steps**.
2. Dot contract defines all three required dimensions: position, fill, animation.
3. Focused task contract defines: method steps, current step, step progress.
4. Task-level visibility states and event payload minimums are documented.
5. Non-goals/guardrails prohibit step-level timeline misuse and ambiguous active-state signaling.

## 9) Out of scope

- visual styling/tokens,
- platform-specific layout or density decisions,
- animation polish/choreography,
- Hive UI implementation details.
