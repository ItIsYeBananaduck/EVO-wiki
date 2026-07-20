---
type: audit-finding
---

> **Archived — Promoted to Lifecycle System**
> - **Lifecycle stage**: spec
> - **Domain**: runtime
> - **Archival date**: 2026-05-12
> - **Archival reason**: Raw note classified and promoted to EVOnotes lifecycle system.
> - **Note**: Canonical/reference copy lives in docs/EVOnotes/spec/runtime/. This file is intake history only.

> **Status: Implementation Artifact**
> Canonical design decision record for shared Alice runtime lifecycle across domain transitions. Source of truth for downstream runtime issues (EVOS1-29, -28, -23, -24). Active doctrine.

# EVOS1-42 Design: Shared runtime ownership and resident Alice lifecycle

> Status: Canonical lifecycle decision record (design-only; no implementation in this issue).
> Scope: Runtime lifecycle, ownership boundaries, app/domain transitions, session handoff, and fallback unload behavior across EVO apps.

## Purpose

This document is the source of truth for how Alice runtime remains resident across app/domain transitions, what state is globally owned versus app-bound, and how fallback behavior must work when residency cannot be preserved.

All downstream runtime implementation issues must conform to this model, especially:

- EVOS1-29 (runtime ownership with `AliceRuntimeManager`)
- EVOS1-28 (DomainBinding overlay switching)
- EVOS1-23 (tool rebinding system)
- EVOS1-24 (session persistence layer)

---

## 1) Core architectural decisions

1. **Single shared runtime host per process**
   - Alice runtime is process-scoped and owned by one runtime host (`RuntimeHostOwner`).
   - Apps/domains attach to the shared host; they do not create competing runtime cores.

2. **Domain/app behavior is overlay-driven, not runtime-recreated**
   - Switching app/domain must primarily rebind overlays (policy/tooling/context adapters) onto one resident core.
   - Runtime recreation is fallback-only.

3. **Strict separation of resident core state from app-bound state**
   - Resident state persists across app switches when feasible.
   - App/domain state is attach-scoped and must be rebindable and disposable.

4. **Lifecycle is state-machine governed**
   - Only explicit transitions are allowed.
   - Invalid transitions are treated as implementation bugs and must emit diagnostics.

5. **Graceful degradation over silent failure**
   - If residency cannot be maintained (memory pressure, host loss, corruption, policy hard-fail), system must follow a defined fallback path with deterministic recovery semantics.

---

## 2) Ownership model and memory boundaries

## 2.1 Resident/shared ownership (process-scoped)

The following are **resident-owned** and persist while runtime remains loaded:

- Runtime core instance (inference engine/process bridge)
- Model/runtime artifacts and heavyweight caches (subject to eviction policy)
- Global scheduler and request queue primitives
- Global observability state (health, counters, runtime IDs)
- Cross-app capability registry metadata (not per-app policy decisions)

## 2.2 App/domain-bound ownership (attach-scoped)

The following are **app/domain-bound** and re-created/rebound per attachment:

- Domain binding overlay (routing + domain policy map)
- Tool capability grants resolved for active app/domain
- App/session context view and scoped memory handles
- UI/session bridge handles and callback channels
- App-local preference overrides and short-lived prompt overlays

## 2.3 Persisted ownership (storage-scoped)

The following are **persisted** and survive process lifetime independently of residency:

- Session summaries/checkpoints defined by EVOS1-24
- Durable metadata required for reattach continuity
- Recovery markers used for crash-safe resume decisions

---

## 3) Canonical lifecycle states

The lifecycle has two planes:

- **Runtime plane** (global resident core)
- **Attachment plane** (active app/domain binding)

## 3.1 Runtime plane states

| State              | Meaning                                                                  | Entry conditions                                                              | Exit conditions                                                                                           |
| ------------------ | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `Uninitialized`    | Runtime host absent.                                                     | Process start or full teardown complete.                                      | `Bootstrapping` on explicit start.                                                                        |
| `Bootstrapping`    | Host creation and runtime init in progress.                              | Start requested from `Uninitialized` or `ColdFallback`.                       | `ResidentIdle` on success, `DegradedFallback` on recoverable failure, `Fatal` on non-recoverable failure. |
| `ResidentIdle`     | Runtime loaded and healthy; no active attachment.                        | Successful bootstrap or detach from `AttachedActive`.                         | `AttachedActive`, `Suspending`, `DegradedFallback`.                                                       |
| `AttachedActive`   | Runtime resident with one active attachment (app/domain).                | Successful attach from `ResidentIdle` or attachment switch from `Rebinding`.  | `Rebinding`, `ResidentIdle` (detach), `Suspending`, `DegradedFallback`.                                   |
| `Rebinding`        | Overlay/tools/context handoff between attachments/domains.               | Requested attach-switch while runtime remains resident.                       | `AttachedActive` on success, `DegradedFallback` on failure.                                               |
| `Suspending`       | Runtime temporarily paused (resource pressure/background policy).        | Suspension requested from idle/active state.                                  | `ResidentIdle` or `AttachedActive` on resume, `ColdFallback` if unload required.                          |
| `ColdFallback`     | Residency lost; runtime core unloaded but recoverable.                   | Forced unload due to pressure, host constraints, or explicit policy fallback. | `Bootstrapping` on restore request.                                                                       |
| `DegradedFallback` | Runtime alive but cannot satisfy normal resident behavior; limited mode. | Recoverable runtime/path failure while host still reachable.                  | `Rebinding`/`AttachedActive` after repair or `ColdFallback` if unload required.                           |
| `Fatal`            | Runtime host unrecoverable in current process.                           | Non-recoverable initialization/runtime failure.                               | No automatic exit; requires process/app-level restart policy.                                             |

## 3.2 Attachment plane states

| State          | Meaning                                                   |
| -------------- | --------------------------------------------------------- |
| `Detached`     | No app/domain currently bound.                            |
| `Attaching`    | Binding/context/tool overlays being established.          |
| `Bound`        | Active app/domain attachment owns foreground interaction. |
| `Detaching`    | Active attachment teardown and handoff in progress.       |
| `AttachFailed` | Attachment-level failure; runtime may still be healthy.   |

Attachment states operate only when runtime is in `ResidentIdle`, `AttachedActive`, or `Rebinding`.

---

## 4) Allowed transitions (hard rules)

## 4.1 Runtime transition rules

Allowed transitions:

- `Uninitialized -> Bootstrapping`
- `Bootstrapping -> ResidentIdle | DegradedFallback | Fatal`
- `ResidentIdle -> AttachedActive | Suspending | DegradedFallback`
- `AttachedActive -> Rebinding | ResidentIdle | Suspending | DegradedFallback`
- `Rebinding -> AttachedActive | DegradedFallback`
- `Suspending -> ResidentIdle | AttachedActive | ColdFallback`
- `DegradedFallback -> Rebinding | AttachedActive | ColdFallback`
- `ColdFallback -> Bootstrapping`

Forbidden transitions (examples):

- `Uninitialized -> AttachedActive` (must bootstrap first)
- `Rebinding -> Uninitialized` (must pass through fallback/teardown)
- `Fatal -> ResidentIdle` (requires external restart path)

## 4.2 Attachment transition rules

- `Detached -> Attaching -> Bound`
- `Bound -> Detaching -> Detached` (detach)
- `Bound -> Detaching -> Attaching -> Bound` (switch)
- Any attach failure must land in `AttachFailed`, then either retry `Attaching` or return to `Detached`.

---

## 5) Runtime host ownership and attach model

1. **RuntimeHostOwner**
   - Single authority for runtime process resources and lifecycle state machine.
   - Owns transition guards, health probes, and fallback escalation.

2. **AttachmentController**
   - Mediates app/domain attach, detach, and switch requests.
   - Enforces serialized handoff semantics (no parallel write-ownership).

3. **DomainBindingResolver**
   - Produces domain overlays for the active attachment.
   - Must be hot-swappable during `Rebinding` without core recreation when healthy.

4. **ToolRebindingCoordinator**
   - Recomputes allowed tool set when binding changes.
   - Must not leak previous app/domain grants after detach/switch.

5. **SessionHandoffBroker**
   - Handles continuity payload handoff between attachments.
   - Works with persistence layer for durable recovery data.

---

## 6) Session handoff expectations

A valid app/domain switch requires:

1. Snapshot outbound session continuation payload from source binding.
2. Detach source binding and revoke source-scoped tool grants.
3. Resolve target binding overlays and target-scoped grants.
4. Bind target context and apply continuity payload according to target policy.
5. Emit handoff completion event with stable request/session correlation IDs.

If any step fails:

- Runtime enters `DegradedFallback` or `AttachFailed` according to the following deterministic mapping:
  - **AttachFailed**: Binding/resolver failures, tool rebinding failures, context-overlay failures, session-payload failures
  - **DegradedFallback**: Runtime health failures, host-path failures, storage-layer failures, driver/core failures
  - **Precedence rule**: When multiple failure categories are observed simultaneously, binding/resolver-related failures trigger `AttachFailed` unless a host runtime-health or driver/core failure is also present, in which case `DegradedFallback` takes precedence (as the runtime itself is impaired, not just the attachment).
- System must not expose a partially switched mixed-binding state.

---

## 7) Unload, suspension, and graceful fallback rules

## 7.1 Suspension triggers

Suspension is allowed for:

- Backgrounding/resource governor policies
- Temporary device thermal/memory pressure
- Explicit platform lifecycle constraints

Suspension requirements:

- Preserve resident-owned metadata where possible.
- Reject new foreground attach-switch operations until resumed.
- Advertise suspend reason and expected resume policy.

## 7.2 Cold fallback (unload) triggers

Cold fallback is required when residency is no longer safe or viable, including:

- Hard memory pressure requiring model/cache release
- Host integrity concerns (runtime corruption indicators)
- Platform-enforced process/resource reclaim

Cold fallback requirements:

- Persist minimum continuity snapshot before unload when possible.
- Emit explicit residency-loss event.
- Next foreground request triggers `Bootstrapping` recovery flow.

## 7.3 Degraded fallback triggers

Degraded fallback is used when runtime is still alive but normal behavior is impaired:

- Binding/tool resolver failure
- Partial dependency outage
- Recoverable internal runtime fault

Degraded fallback requirements:

- Limit operations to safe subset.
- Prefer in-place recovery (`Rebinding`) before cold restart.
- Escalate to `ColdFallback` if recovery budget exceeded.

---

## 8) Invariants (must always hold)

1. At most one active foreground attachment owns interactive runtime use at a time.
2. Resident core identity remains stable across successful `Rebinding` transitions.
3. App/domain-scoped grants are revoked before new grants are applied on switch.
4. No request may execute against a stale/detached binding.
5. Every fallback event must be observable with reason, state, and recovery outcome.
6. Persistence writes for continuity must be idempotent for retry safety.

---

## 9) Downstream implementation constraints

## 9.1 EVOS1-29 (`AliceRuntimeManager`)

Must implement:

- Runtime plane state machine above as explicit enum + guarded transition API
- Host ownership singleton semantics
- Fallback escalation paths (`DegradedFallback` vs `ColdFallback`)

## 9.2 EVOS1-28 (DomainBinding overlay switching)

Must implement:

- `Rebinding` transition contract with no core recreation on healthy path
- Switch atomicity (no mixed source/target overlays)
- Failure routing into `AttachFailed`/`DegradedFallback` according to the following deterministic mapping:
  - **AttachFailed**: Binding/resolver failures, tool rebinding failures, context-overlay failures, session-payload failures
  - **DegradedFallback**: Runtime health failures, host-path failures, storage-layer failures, driver/core failures
  - **Precedence rule**: When multiple failure categories are observed simultaneously, binding/resolver-related failures trigger `AttachFailed` unless a host runtime-health or driver/core failure is also present, in which case `DegradedFallback` takes precedence (as the runtime itself is impaired, not just the attachment).

## 9.3 EVOS1-23 (tool rebinding)

Must implement:

- Deterministic revoke-then-grant during switch
- Attach-scoped tool grant ownership
- Guardrails preventing stale grant usage post-detach

## 9.4 EVOS1-24 (session persistence)

Must implement:

- Continuity snapshot schema compatible with handoff flow
- Pre-unload persistence checkpoint hooks
- Recovery-path resume semantics for `ColdFallback -> Bootstrapping`

---

## 10) Acceptance criteria mapping

- **Clear resident runtime lifecycle model exists:** ✅ Defined runtime and attachment state machines with explicit semantics.
- **Shared vs app-specific state is defined:** ✅ Resident/app/persisted ownership boundaries are explicit.
- **Transition and fallback rules are explicit:** ✅ Allowed transitions, forbidden transitions, and fallback triggers are documented.
- **Downstream implementation can proceed without ambiguity:** ✅ Per-issue implementation constraints are enumerated.

---

## 11) Out of scope for EVOS1-42

- Concrete API signatures and code-level interface definitions (covered by related design/implementation issues).
- Platform-specific process-lifecycle plumbing details.
- Performance tuning and cache sizing heuristics.