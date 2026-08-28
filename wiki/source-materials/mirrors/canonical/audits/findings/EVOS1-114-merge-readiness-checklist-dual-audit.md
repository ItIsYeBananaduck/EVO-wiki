---
title: "EVOS1-114 — Merge-Readiness Checklist and Blocker List (Dual-Audit Synthesis)"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-03-31
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-114 — Merge-Readiness Checklist and Blocker List (Dual-Audit Synthesis)

**Date**: 2026-03-31  
**Issue**: EVOS1-114  
**Inputs synthesized**:

- EVOS1-111 structural migration audit (`main..testing`) from `docs/audits/PRE_MERGE_SANITY_AUDIT_TESTING_TO_MAIN.md`
- EVOS1-113 migration-wave diff audit (pre-migration baseline → testing) from `docs/audits/EVOS1-113-migration-wave-diff-audit-prebaseline-to-testing.md`

---

## 1) Synthesis outcome

The two source audits agree on a key point: migration/extraction structure is largely sound, but merge readiness is **not unconditional**.

- EVOS1-111 flags branch reconciliation and Alice conflict review as required before merge.
- EVOS1-113 flags behavior-level risks (approval-state logic, runtime ownership lifecycle, orchestration fallback correctness) that require focused verification before calling the branch safe.

This document converts those findings into a single actionable go/no-go checklist.

---

## 2) Critical blockers (must be resolved before merge)

These are **no-go** items. Any open item below blocks merge.

1. **Branch divergence reconciliation is incomplete**
   - `main` and `testing` contain non-shared commit chains; merge/rebase coordination must be completed.
   - Required evidence: reconciliation commit(s) present and conflicts fully resolved.

2. **High-risk behavior paths not manually verified**
   - Approval lifecycle semantics (`needs_approval`/`approved` with `blocked` interpretation).
   - Runtime ownership/disposal semantics when orchestrator is injected with external `TaskRuntime`.
   - Required evidence: test notes/logs for both flows with pass/fail outcomes.

3. **Alice presentation merge integrity not confirmed after reconciliation**
   - Re-export shim pattern must remain intact in `flutter_app/lib/features/alice/presentation/*` where intended.
   - Required evidence: post-reconcile file review confirming no accidental regressions.

4. **Build/package dependency resolution not validated post-reconcile**
   - `flutter_app` and `apps/evo_connect` dependency resolution must succeed on reconciled branch.
   - Required evidence: successful `flutter pub get` in both app roots.

---

## 3) Medium-risk verification items (required for proceed-to-testing)

These do not immediately imply a hard merge halt, but they are required to move from “blocked” to “proceed to testing”.

1. **Hive orchestration fallback route correctness**
   - Validate candidate ranking/fallback does not silently route critical tasks to under-capable nodes.

2. **Training executor catalog assertion robustness**
   - Confirm debug/release startup behavior with current catalog.
   - Run one controlled catalog-size drift simulation to ensure failure mode is explicit and expected.

3. **Shim/import normalization visibility**
   - Enumerate remaining imports that still consume app-local Alice shim files.
   - Confirm migration intent (temporary compatibility vs final state) is explicit.

4. **Baseline reproducibility note alignment**
   - EVOS1-113 baseline SHA correction is documented; confirm downstream references use reachable SHA.

---

## 4) Low-risk deferrable items (can be scheduled after merge if all blockers clear)

1. **Complete Talent card UI extraction decision**
   - Decide whether `talent_task_card.dart` remains app-specific or is package-promoted.

2. **Shim deprecation cleanup**
   - Remove transition-era Alice shims once direct package imports are complete.

3. **Governance/documentation hardening from EVOS1-89 lineage**
   - Canonical artifact model clarity, typed discriminator improvements, operational reference cleanup.

---

## 5) Build checklist (post-reconcile)

- [ ] Reconcile `main` and `testing` histories.
- [ ] Resolve all merge conflicts (especially Alice surfaces and docs).
- [ ] Regenerate/refresh lockfiles if required by the chosen merge strategy.
- [ ] Run package-level tests:
  - [ ] `packages/delegator` tests
  - [ ] `packages/ai-runtime` tests
  - [ ] `packages/hive` tests
- [ ] Confirm CI pipeline is green on reconciled branch.

---

## 6) Package validation checklist

- [ ] `packages/core` re-export façade remains intentional and points to correct extracted implementations.
- [ ] `packages/delegator` policy contracts and public API surface unchanged unintentionally.
- [ ] `packages/hive` orchestration/task-execution contracts resolve and integrate cleanly.
- [ ] `packages/ui` exports include required Alice/components entry points.
- [ ] `packages/theme` token exports remain stable.
- [ ] `packages/domains/training` executor catalog contracts match expected runtime wiring.

---

## 7) App startup checklist

- [ ] `flutter_app`: `flutter pub get` succeeds.
- [ ] `apps/evo_connect`: `flutter pub get` succeeds.
- [ ] Debug startup succeeds with no assertion failures from executor registration.
- [ ] Release/profile startup sanity check succeeds.
- [ ] Core task/orchestrator initialization produces expected runtime attachment behavior.

---

## 8) High-risk manual flows (must-pass)

1. **Delegator approval lifecycle flow**
   - Create approval-required task → verify blocked state.
   - Approve task → verify unblock and execution.
   - Deny/cancel path → verify consistent state/log behavior.

2. **Orchestrator runtime ownership flow**
   - Case A: orchestrator-created runtime.
   - Case B: externally injected shared runtime.
   - In both: verify no double-dispose, no leaked listeners, expected event delivery.

3. **Hive route selection flow**
   - Full capability match scenario.
   - Partial/fallback scenario.
   - Verify route reason and outcome correctness.

4. **Alice UI parity flow**
   - Compare package-backed rendering against expected baseline behavior (visual/interaction parity).

5. **Training executor registration flow**
   - Validate current catalog path.
   - Validate intentionally perturbed catalog-size path for explicit failure semantics.

---

## 9) Go / No-Go criteria

## Proceed to testing (GO to next phase)

All of the following are true:

- Critical blockers are closed.
- Medium-risk verification items have explicit pass evidence.
- Build checklist and app startup checklist are fully green.
- High-risk manual flows pass.

## Fix blockers first (NO-GO for merge, GO for additional remediation)

Use this disposition when:

- Any critical blocker remains open, or
- High-risk manual flows are untested or failing.

## Halt merge (HARD NO-GO)

Use this disposition when:

- Reconciliation introduces unresolved conflicts or unstable build state, or
- Behavior regressions are confirmed in approval/runtime ownership/orchestration flows without an accepted mitigation.

---

## 10) Final merge recommendation (current state)

**Recommendation: FIX BLOCKERS FIRST.**

Rationale:

- Dual-audit evidence indicates structurally healthy migration patterns, but merge-critical prerequisites (branch reconciliation + behavior-critical verification evidence) are not yet fully satisfied in the synthesized state.
- Once section 2 blockers are cleared and section 8 manual flows pass, disposition can be upgraded to **Proceed to testing**.