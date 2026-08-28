---
title: "EVOC-215 — Timeline Visualization and Density Rules"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-04-02
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-215 — Timeline Visualization and Density Rules

> Status: Draft contract (architecture only; no UI implementation).
> Date: 2026-04-02.
> Parent: EVOC-210 — Connect Core Interaction Contract.

## 1) Purpose

Define the timeline visualization and density contract for multi-agent task orchestration so crowded execution states remain understandable, interactive, and semantically consistent across surfaces.

This contract covers:

- dot-based timeline representation for task units,
- overlap and stacking behavior when tasks share time/index density,
- cluster summarization with `+X` overflow indicator,
- expand behavior for stacked clusters,
- active-focus exclusivity,
- and explicit non-goals.

## 2) Core invariants

1. A timeline dot represents exactly one task.
2. The timeline can encode multi-agent orchestration, but task identity must remain distinguishable within dense clusters.
3. Full visual overlap is prohibited.
4. Partial overlap is allowed only under controlled stacking rules.
5. A cluster may show at most 4 visible dots; any remainder is summarized as `+X`.
6. Tapping a stacked cluster must expand it into a distinguishable task list/state.
7. Only one active focus task may exist at a time per execution stream.

## 3) Timeline density model

### 3.1 Dot representation

- Each task is rendered as one dot on the timeline.
- Dot ordering follows the canonical execution ordering contract from EVOC-213.
- Multi-agent assignment may influence dot metadata (for example, owner badge/icon), but not the one-dot-per-task invariant.

### 3.2 Overlap policy

Definitions:

- **Full overlap**: one dot completely occludes another such that one or more tasks become undiscoverable.
- **Partial overlap**: controlled adjacency/stacking where multiple dots are still perceptible as separate task tokens.

Rules:

1. Full overlap is not allowed under any density condition.
2. Partial overlap is allowed only when all visible dots remain distinguishable.
3. Distinguishability must not rely on color alone; at least one additional differentiator (position offset, outline, badge, or label marker) is required.

### 3.3 Controlled stacking limits

For any timeline position/bucket where more than one task competes for the same visual slot:

1. Show up to **4 visible dots** (platform may choose 3 or 4 as long as the choice is consistent within a surface).
2. If total tasks exceed visible capacity, render a summary indicator:
   - format: `+X`
   - where `X = totalTasksInCluster - visibleDots`
3. Visible dots in a stack must preserve stable ordering, preferably by execution order then timestamp.

## 4) Cluster interaction contract

When a user taps/clicks a stacked cluster (including `+X`):

1. The system must expand the cluster into a view that exposes all tasks in that cluster.
2. Expanded items must be individually selectable.
3. Expanded view must preserve per-task distinguishability and state visibility.
4. Expansion must not imply task completion, approval, or re-ordering.
5. Collapse behavior should return to the compact timeline without loss of state.

## 5) Active focus exclusivity

1. At most one task can be the **active focus task** at a time per execution stream.
2. Focus transfer from task A to task B must be explicit and atomic:
   - A exits focus,
   - B enters focus,
   - no simultaneous dual-focus state is valid.
3. Cluster expansion does not override the single-focus invariant; it only changes visibility granularity.

## 6) Event contract (density + cluster behavior)

Required event types:

- `timeline_density.cluster_rendered`
- `timeline_density.cluster_overflowed`
- `timeline_density.cluster_expanded`
- `timeline_density.cluster_collapsed`
- `timeline_density.focus_changed`

Minimum fields:

- `executionId`
- `clusterId` (nullable for non-cluster events)
- `taskIds` (array; for cluster events)
- `visibleDotCount`
- `overflowCount`
- `totalClusterTasks`
- `focusedTaskId` (nullable)
- `timestamp`
- `surface`

Additional constraints:

- `overflowCount` must equal `max(0, totalClusterTasks - visibleDotCount)`.
- `focusedTaskId` must reference exactly one task when non-null.

## 7) Acceptance criteria

This issue is complete when:

1. Timeline unit is defined as task-level dot representation.
2. Multi-agent orchestration is explicitly supported in timeline semantics.
3. Full overlap prohibition and partial-overlap allowance are both documented.
4. Controlled stacking limit (3–4 visible dots max) and `+X` overflow behavior are defined.
5. Distinguishability requirements for stacked dots are explicit.
6. Tap-to-expand behavior for stacked clusters is defined.
7. Single active focus task rule is explicit and testable.

## 8) Out of scope

- visual styling tokens/themes,
- final animation choreography,
- platform-specific component implementations,
- timeline virtualization/performance internals.