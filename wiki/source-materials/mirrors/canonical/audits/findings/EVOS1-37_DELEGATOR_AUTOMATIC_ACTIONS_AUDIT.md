---
type: audit-finding
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-37 — Audit automatic actions for Delegator governance

Date: 2026-03-28  
Owner: EVOsystem / runtime governance

## Scope

This audit covers the 9 automatic actions currently defined in `training/enf_lora/alice_capability_map.json` under `automaticActions`, plus governance treatment for vision capabilities and the `navigate` action.

## Decision framework

Each automatic action is classified as one of:

- **governed**: must pass through Delegator policy evaluation and (where applicable) user approval callbacks before side effects are applied.
- **explicitly-exempt**: bypasses user approval but still emits Delegator audit metadata and can be disabled by policy/versioned config.

## Automatic actions inventory and classification (all 9)

| Automatic action key          | Current behavior summary                                             | Classification        | Governance rationale                                                                                                          |
| ----------------------------- | -------------------------------------------------------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `strainBasedRestAdjustment`   | Auto-adjusts live rest interval based on strain score during workout | **explicitly-exempt** | Low-risk, reversible session UX adjustment; keep real-time execution unblocked but require Delegator telemetry + kill switch. |
| `weekOverWeekProgression`     | Proposes weekly progression updates to workout plan                  | **governed**          | Persistent program mutation and overload risk; must require Delegator policy + user approval.                                 |
| `mesocycleTransition`         | Starts next mesocycle with updated progression type/length           | **governed**          | Long-horizon training-plan mutation; requires explicit authorization path and audit.                                          |
| `exerciseReplacement`         | Replaces low-compliance exercise with alternative                    | **governed**          | Changes exercise prescription and movement selection; requires policy + confirmation.                                         |
| `volumeAdjustment`            | Applies post-session volume guardrail changes                        | **governed**          | Directly changes future load/volume targets; requires a governed path before persistence.                                     |
| `systemDeload`                | Triggers deload stop/scheduling from readiness or duration           | **governed**          | Safety-critical intervention with training availability impact; must be Delegator-mediated.                                   |
| `alicePerceivedIntensity`     | Computes/stores intensity signal for analytics blending              | **explicitly-exempt** | Data enrichment only, no direct user-visible mutation; allow auto path with audit tagging.                                    |
| `musicPerformanceCorrelation` | Computes track-performance correlations                              | **explicitly-exempt** | Offline/analytic inference with no immediate destructive side effect; log-only governance envelope.                           |
| `performancePlaylistCreation` | Creates performance-optimized playlist recommendations               | **explicitly-exempt** | Non-safety personalization output; no critical training-state mutation. Keep policy-controlled and user-overridable.          |

## Required Delegator policy controls for automatic actions

For all 9 automatic actions:

1. Emit `delegator_contract_version`, `policy_version`, `action_key`, `classification`, `decision`, and `execution_mode` in audit records.
2. Support deny-by-policy at runtime through a centralized rule map (including all explicitly-exempt actions).
3. Preserve deterministic replay: same inputs + policy version must produce the same allow/deny outcome.

Additional requirements for **governed** actions:

- Must evaluate Delegator task actionability before any side effect.
- Must invoke approval callback when action requires confirmation.
- Must fail closed when approval handler is unavailable.

Additional requirements for **explicitly-exempt** actions:

- No user approval callback required by default.
- Must still be individually disableable by policy and globally disableable via emergency switch.

## Vision tools approval flow (designed)

Vision capabilities in scope: `poseEstimator`, `ocr`, `bodyCompositionScan`, `csvPlanImport`, `ocrFallback`.

### Flow design

1. **Capability preflight (Delegator)**
   - Validate camera/file permission state + surface context.
   - Confirm tool is allowed by policy tier.

2. **Tiered approval policy**
   - **Tier V0 (exempt, session-scoped):** `poseEstimator` during active live workout when permission already granted.
   - **Tier V1 (single-confirm or remembered consent):** `ocr`, `ocrFallback`, `csvPlanImport`.
   - **Tier V2 (always governed + explicit confirmation):** `bodyCompositionScan` because health/body metrics are sensitive.

3. **Execution + audit envelope**
   - Log capability, consent mode, input origin (camera/file), and output destination.
   - Reject execution if consent/policy preconditions fail.

4. **Post-action transparency**
   - User-facing notice for extracted/imported data and where it was stored.
   - One-tap revoke for remembered consent at settings level.

## Navigation action governance decision

Action: `agenticActions.navigate` currently marked as always allowed.

### Decision

- Classify `navigate` as **explicitly-exempt** from approval prompts.
- Keep `navigate` **inside Delegator policy scope** (not a true bypass) so it can be blocked in sensitive contexts (e.g., modal safety flows, locked onboarding steps).

### Policy constraints

- Allow by default for user-requested route changes.
- Deny when route transition would interrupt an active safety-critical flow.
- Emit audit records for denied route attempts and policy reasons.

## Acceptance criteria check

- [x] All 9 automatic actions documented.
- [x] Each action classified as governed or explicitly-exempt.
- [x] Vision tools approval flow designed.
- [x] Navigation action governance decision made.