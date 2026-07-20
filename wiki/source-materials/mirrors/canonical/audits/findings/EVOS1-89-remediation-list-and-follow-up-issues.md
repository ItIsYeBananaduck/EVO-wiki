---
type: audit-finding
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-89 — Remediation List and Ordered Follow-up Issues from Audit Findings

Date: 2026-03-31  
Parent stream: EVOS1-83 post-merge audit  
Input audits: EVOS1-85, EVOS1-86, EVOS1-87, EVOS1-88, EVOS1-90, EVOS1-91, EVOS1-93, EVOS1-94, EVOS1-96

---

## Executive summary

- EVOS1-85 through EVOS1-88 found **no source-surface regressions** in the scoped diff (`9cd1b059..556b5976`) because the diff was documentation-only.
- EVOS1-90/93/94/96 identified a **governance and architecture clarity gap**: active implementation is single-lane `global_patch`, while GU/GT split semantics are still conceptual.
- The highest-risk items are governance blockers that must be resolved before downstream polish/documentation cleanup.

---

## Severity model used

- **Critical (Blocker):** prevents safe/correct delivery decisions or can cause teams to build against the wrong contract.
- **High:** material risk to auditability, operability, or migration safety; should be handled immediately after critical blockers.
- **Medium:** important cleanup that reduces drift and prevents future misimplementation.
- **Low (Polish):** non-blocking clarity improvements.

---

## Consolidated findings by severity

## Critical (Blockers)

1. **Canonical artifact contract mismatch (implemented vs implied):**
   - Implemented path publishes one artifact lane (`global_patch_metadata`), while docs and planning language often imply first-class GU/GT lanes.
   - Risk: downstream work can implement incompatible assumptions.
   - Sources: EVOS1-93, EVOS1-94, EVOS1-96.

## High

2. **No typed artifact discriminator in active schema/API contract:**
   - Current metadata and publish endpoint do not encode `artifact_family` (GU/GT).
   - Risk: cannot safely evolve to multi-family publication without ambiguity.
   - Sources: EVOS1-91 target definition gap vs EVOS1-94 current-state audit.

3. **Legacy/dormant Fly references create operational ambiguity:**
   - Active merge/publish path is hosting-agnostic, but legacy references and placeholder endpoints remain.
   - Risk: teams may target non-canonical paths during operations/incidents.
   - Sources: EVOS1-90.

## Medium

4. **Docs and implementation terminology drift:**
   - Multiple docs describe GU/GT behaviors as if already active.
   - Risk: governance review and implementation tickets inherit incorrect assumptions.
   - Sources: EVOS1-91/93/94.

5. **Client-family retrieval policy not yet codified for future typed lanes:**
   - Device/server boundary is clear, but family-aware retrieval and fallback policy is not yet formalized in runtime contracts.
   - Risk: inconsistent client rollout when typed lanes are introduced.
   - Sources: EVOS1-91, EVOS1-96.

## Low (Polish)

6. **Scoped audit range was docs-only:**
   - EVOS1-85/86/87/88 passed with no runtime/package/UI/entrypoint changes.
   - Improvement: capture a new scope whenever source surfaces change so these audits can validate real code paths.
   - Sources: EVOS1-85 through EVOS1-88.

---

## Ordered remediation backlog (blockers before dependents)

## Critical fixes (execute first)

1. **[Blocker] Decide and record canonical near-term artifact truth**
   - **Proposed issue title:** `Formalize canonical artifact model for federation (single-lane now vs typed GU/GT target)`
   - **Deliverable:** explicit governance decision log that states what is true today and what is target state.
   - **Depends on:** none.

2. **[Blocker] Align all federation docs to canonical truth**
   - **Proposed issue title:** `Normalize federation docs to current canonical artifact lane and clearly mark GU/GT as target`
   - **Deliverable:** remove ambiguous language implying active GU/GT split where not implemented.
   - **Depends on:** #1.

## High-priority follow-ups

3. **Introduce typed artifact metadata contract (backward-compatible)**
   - **Proposed issue title:** `Add artifact_family/domain/version typing to federation metadata and API contract`
   - **Deliverable:** schema and API additions with compatibility bridge from `global_patch`.
   - **Depends on:** #1.

4. **Implement typed publication lanes for GU and GT in orchestration**
   - **Proposed issue title:** `Implement family-scoped aggregation/publication jobs for GU and GT`
   - **Deliverable:** independent `(domain, family)` version lanes with lifecycle controls.
   - **Depends on:** #3.

5. **Deprecate or remove dormant Fly placeholder wiring**
   - **Proposed issue title:** `Retire dormant Fly placeholders and mark infrastructure references as historical or active`
   - **Deliverable:** cleanup of placeholder endpoint usage and explicit operational runbook notes.
   - **Depends on:** #2 (documentation alignment).

## Cleanup and polish

6. **Codify client retrieval/rollback policy for typed families**
   - **Proposed issue title:** `Define client runtime policy for family-aware retrieval, verification, fallback, and rollback`
   - **Deliverable:** runtime contract and rollout guardrails for family-aware activation.
   - **Depends on:** #3.

7. **Create recurring “source-change-triggered” audit guardrail**
   - **Proposed issue title:** `Add audit trigger rule: rerun EVOS1-85..88 only when code surfaces are touched`
   - **Deliverable:** governance checklist update reducing false “pass/no-op” cycles on docs-only diffs.
   - **Depends on:** none.

---

## Dependency chain (explicit)

- **Critical path:** `#1 -> #2 -> #5` and `#1 -> #3 -> #4` with `#3 -> #6`.
- **Independent governance improvement:** `#7`.

This ordering places blockers before dependents and separates immediate risk reduction from cleanup/polish.

---

## Acceptance criteria mapping (EVOS1-89)

- **Findings categorized by severity:** completed (Critical/High/Medium/Low sections).
- **Follow-up issues proposed in correct order:** proposed ordered backlog with explicit dependencies.
- **Critical blockers identified clearly:** completed (items #1 and #2 flagged as blockers, with #1 as root blocker).