---
title: "EVOS1-113 — Migration-Wave Diff Audit (Pre-Migration Baseline → Current testing)"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-03-31
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-113 — Migration-Wave Diff Audit (Pre-Migration Baseline → Current testing)

**Date**: 2026-03-31  
**Issue**: EVOS1-113  
**Scope baseline (from EVOS1-112 intent)**: pre-migration testing baseline before PR #152  
**Baseline used for reproducible diff**: `9de4ce1c9dc1978bb2b8d2e01581289135faabd0`  
**Current testing tip analyzed**: `05bd72b599a999b0a0ccbf3906aa7b112f76b497`  
**Primary diff range**: `9de4ce1c9dc1978bb2b8d2e01581289135faabd0..05bd72b599a999b0a0ccbf3906aa7b112f76b497`

> Note: EVOS1-112 documented baseline SHA as `9de4ce1c2d84553bc4ebc665617f4ee1ebed14f6`, which does not exist in repo history. The reachable commit is `9de4ce1c9dc1978bb2b8d2e01581289135faabd0`.

---

## 1) Scope and change volume (targeted surfaces only)

Focus areas audited:

- `packages/core`
- `packages/ai-runtime`
- `packages/delegator`
- `packages/hive`
- `packages/ui`
- `packages/theme`
- `packages/tools`
- `packages/domains/training`
- `packages/mesh`
- `flutter_app`
- `apps/evo_connect`

### Aggregate in-scope diff

- **87 files changed**
- **+9,837 / -1,860 lines**

### Per-area change density

- `packages/ai-runtime`: 16 files, +3,653 / -0
- `packages/delegator`: 7 files, +623 / -0
- `packages/domains/training`: 6 files, +1,759 / -0
- `packages/hive`: 14 files, +791 / -12
- `packages/ui`: 15 files, +1,978 / -50
- `packages/tools`: 3 files, +233 / -1
- `packages/core`: 5 files, +92 / -117
- `packages/theme`: 7 files, +98 / -30
- `apps/evo_connect`: 6 files, +571 / -30
- `flutter_app`: 8 files, +39 / -1,620
- `packages/mesh`: no changes

Interpretation: this wave is predominantly a **modular extraction + package promotion** event with largest net additions in new packageized surfaces and net deletions in `flutter_app`.

---

## 2) Pure extraction vs behavior change

## Mostly extraction / boundary moves

These patterns are predominantly structural extraction:

1. **Core delegator moved to dedicated package**
   - `packages/core/lib/src/delegator/*` now acts as re-export facade into `evo_delegator`.
   - New implementation and contracts live in `packages/delegator`.

2. **UI primitives and Alice rendering extracted to `packages/ui` + `packages/theme`**
   - New package-level Alice widgets/primitives added (`alice_hologram_projection`, `alice_png_avatar`, layout primitives, chips, status dot, etc.).
   - Flutter app local Alice files now mostly thin exports back to package implementations.

3. **Training domain surfaced as package**
   - `packages/domains/training` introduces executor catalog/context mapping and domain entrypoint.

4. **Hive orchestration/task execution extracted to package-level modules**
   - New orchestration and task-execution surfaces introduced in `packages/hive`.

5. **Tool interaction layer extracted**
   - `packages/tools/lib/src/tool_interactive.dart` added.

## Behavior changes (not just movement)

At least these deltas are behavioral and require runtime validation:

1. **Task approval blocking semantics changed** (`packages/core/lib/src/runtime/task_runtime.dart`)
   - Old logic: looked at latest `needs_approval` log only.
   - New logic: scans reverse logs for latest state among `{needs_approval, approved}` and only blocks if latest state is `needs_approval && blocked == true`.
   - Impact: tasks with stale historical `needs_approval` entries but later `approved` entries now unblock correctly; any metadata inconsistency may alter execution gating.

2. **Orchestrator runtime ownership lifecycle changed** (`apps/evo_connect/lib/services/orchestrator.dart`)
   - Constructor now supports injected `TaskRuntime`.
   - Disposal only disposes runtime if orchestrator owns it.
   - Impact: fixes double-disposal/leaked ownership patterns, but introduces dependency-injection lifecycle contract that can regress if callers assume old always-own behavior.

3. **Executor registration adds strict assertion** (`flutter_app/lib/features/alice/domain/action_runtime/executors/executor_registry.dart`)
   - Adds `assert(TrainingExecutorCatalog.all.length == 3);`.
   - Impact: debug-mode startup coupling to training catalog cardinality; future catalog evolution can fail assertions unless updated intentionally.

---

## 3) Duplicate or stale logic still present

1. **Transition-era dual-location Alice surfaces remain**
   - `flutter_app/lib/features/alice/presentation/alice_hologram_projection.dart`
   - `flutter_app/lib/features/alice/presentation/alice_png_avatar.dart`
   - These are now export shims to `evo_ui`, which is acceptable for migration compatibility but should be tracked for deprecation/removal once imports are fully normalized.

2. **UI migration incomplete for Talent card**
   - `flutter_app/lib/features/alice/presentation/talent_task_card.dart` still houses substantial UI logic and is not yet package-promoted like other Alice presentation surfaces.
   - Suggestion: either deliberately keep app-specific or complete extraction for consistency.

3. **Baseline reference drift in EVOS1-112 artifact**
   - Documented baseline SHA typo/nonexistent hash can cause audit reproducibility failures if reused mechanically.

---

## 4) Weak boundaries still observed

1. **Core package as migration facade**
   - `packages/core` currently re-exports extracted package logic (`evo_delegator`) rather than fully shedding ownership semantics. Boundary is improved but still transitional.

2. **App/runtime coupling persists in orchestration flow**
   - `apps/evo_connect` still owns significant orchestration behavior (chat/task attachment lifecycle) while depending on extracted runtime/task packages.
   - Boundary contracts exist, but correctness still depends on app-side sequencing.

3. **Debug assertions as implicit contract enforcement**
   - Cardinality assert in executor registry indicates coupling to fixed training catalog assumptions rather than declarative capability negotiation.

---

## 5) What compiles but may behave incorrectly (regression candidates)

High-confidence regression candidates to prioritize:

1. **Approval state edge-cases in task runtime**
   - Mixed or missing `chatState` metadata across legacy logs may produce incorrect unblock behavior.

2. **Injected runtime lifecycle in orchestrator**
   - Shared runtime instances across screens/components can leak listeners or be disposed unexpectedly if ownership assumptions diverge.

3. **Hive orchestration policy fallback correctness**
   - Capability matching policy falls back to best-score candidate; verify no silent routing to under-capable nodes for critical tasks.

4. **Training executor catalog drift vs assertion**
   - Expansion/reduction of training executors can fail debug builds or mask intended optional executor behavior.

5. **Package extraction + shim consistency risk**
   - Coexistence of app shims and package implementations can hide stale imports and delay true boundary closure.

---

## 6) Top migration risks introduced

1. **Contract drift between extracted packages and app orchestration** (High)
2. **Approval/blocked-state logic misclassification under historical logs** (High)
3. **Partial extraction leaving stale/duplicate integration paths** (Medium)
4. **Reproducibility risk from incorrect baseline SHA in prior artifact** (Medium)
5. **Inference routing under-constrained by best-effort capability matching** (Medium)

---

## 7) Manual verification targets (merge decision checklist inputs)

1. **Delegator approval lifecycle**
   - Create task requiring approval → approve/deny transitions → verify block/unblock and logs.

2. **Orchestrator runtime ownership**
   - Test with internally created runtime and injected shared runtime; confirm no double-dispose and event stream correctness.

3. **Hive orchestration route selection**
   - Validate candidate ranking and fallback route reason for full-match vs partial-match capability sets.

4. **Alice UI package extraction parity**
   - Compare package-based `AlicePngAvatar`/`AliceHologramProjection` behavior with prior app-local rendering (animation timing, visual offsets, interaction).

5. **Training executor registration path**
   - Verify startup behavior in debug/release with current executor catalog and one simulated catalog-size change.

6. **Shim deprecation readiness**
   - Enumerate import graph for `flutter_app/.../alice_*` shim files and plan cutover to direct `evo_ui` imports.

---

## 8) Merge-readiness disposition for EVOS1-113

- **Migration diff analyzed independently**: ✅ (baseline→testing targeted diff complete)
- **Key risks identified**: ✅
- **Findings usable for merge decision**: ✅

Recommendation: proceed to EVOS1-114 checklist synthesis with explicit gating on the regression candidates above, especially approval-state semantics and runtime ownership lifecycle.

---

## Repro commands

```bash
# verify audited commit endpoints (fail fast if either SHA missing)
git cat-file -e 9de4ce1c9dc1978bb2b8d2e01581289135faabd0 || { echo "Error: baseline SHA 9de4ce1c9dc1978bb2b8d2e01581289135faabd0 not found"; exit 1; }
git cat-file -e 05bd72b599a999b0a0ccbf3906aa7b112f76b497 || { echo "Error: testing tip SHA 05bd72b599a999b0a0ccbf3906aa7b112f76b497 not found"; exit 1; }

# full in-scope delta summary
git diff --shortstat 9de4ce1c9dc1978bb2b8d2e01581289135faabd0..05bd72b599a999b0a0ccbf3906aa7b112f76b497 -- \
  packages/core packages/ai-runtime packages/delegator packages/hive \
  packages/ui packages/theme packages/tools packages/domains/training \
  packages/mesh flutter_app apps/evo_connect

# changed files with rename detection
git diff --name-status -M 9de4ce1c9dc1978bb2b8d2e01581289135faabd0..05bd72b599a999b0a0ccbf3906aa7b112f76b497 -- \
  packages/core packages/ai-runtime packages/delegator packages/hive \
  packages/ui packages/theme packages/tools packages/domains/training \
  packages/mesh flutter_app apps/evo_connect

# per-area shortstats
for p in packages/core packages/ai-runtime packages/delegator packages/hive packages/ui packages/theme packages/tools packages/domains/training packages/mesh flutter_app apps/evo_connect; do
  echo "## $p"
  git diff --shortstat 9de4ce1c9dc1978bb2b8d2e01581289135faabd0..05bd72b599a999b0a0ccbf3906aa7b112f76b497 -- "$p"
done
```