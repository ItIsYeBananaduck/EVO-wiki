---
title: "EVOS1-21 Design: IPC strategy for future multi-app separation"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-05-12
---

> **Archived — Promoted to Lifecycle System**
> - **Lifecycle stage**: spec
> - **Domain**: runtime
> - **Archival date**: 2026-05-12
> - **Archival reason**: Raw note classified and promoted to EVOnotes lifecycle system.
> - **Note**: Canonical/reference copy lives in docs/EVOnotes/spec/runtime/. This file is intake history only.

> **Status: Implementation Artifact**
> IPC strategy for future multi-app domain separation. Design-only (migration-first). Still the active design contract for cross-app boundaries. Not yet implemented — remains forward-looking doctrine.

# EVOS1-21 Design: IPC strategy for future multi-app separation

_Date_: 2026-03-30  
_Status_: Proposed (architecture decision for future domain app split)

---

## 1) Scope and intent

This design defines how EVO should communicate across apps/processes if domains (`mind`, `learn`, `connect`, `training`) are later split into separate app bundles.

The document covers:

- platform IPC options and tradeoffs,
- recommendation for a preferred architecture,
- behavior for in-flight inference during app switches,
- Delegator approval UI ownership model in cross-app scenarios,
- migration path from current single-process runtime.

This design is intentionally migration-first: it preserves today’s single-process architecture while introducing seams for future process/app boundaries.

---

## 2) Assumptions and design constraints

1. **Today = single-process host**
   - Current runtime assumptions remain valid; no forced split in this issue.

2. **Future = per-domain app possibility**
   - Any domain may become a separate app depending on product/GTM and policy needs.

3. **Safety > convenience**
   - Delegator approval and tool governance must remain deterministic across process boundaries.

4. **No hard dependency on continuous background execution**
   - IPC model must tolerate suspension, process death, and delayed delivery.

5. **Cross-platform parity at contract level**
   - iOS and Android implementations can differ internally, but message contracts and orchestration semantics should stay shared.

---

## 3) IPC options evaluated

## 3.1 iOS option A: App Extension + App Group shared container

### Summary

Use app extensions for entry points/limited delegated operations and App Group shared storage for durable handoff payloads.

### Pros

- Native Apple-supported app-to-extension and host-to-extension model.
- App Group container enables low-friction shared state and queue persistence.
- Good fit for short, user-triggered operations and continuity payload exchange.

### Cons

- Extension lifetime and resources are constrained; unsuitable as long-running inference host.
- Serialization/compat overhead when exchanging complex runtime state.
- Requires strong idempotency because extension execution may be interrupted.

### Best fit

- Request envelopes, approval payloads, handoff intents, resume tokens, and short bounded operations.

---

## 3.2 iOS option B: XPC service boundary

### Summary

Use an XPC-style process boundary for stricter service isolation and protocol-based IPC.

### Pros

- Clean interface contracts and process isolation semantics.
- Stronger decoupling between UI process and runtime/service ownership.
- Good long-term shape for a broker model.

### Cons

- Platform feasibility for iOS app-store-distributed multi-app runtime services is materially constrained compared with macOS.
- Operational complexity is higher than extension + shared container.
- Requires robust reconnect/session recovery handling.

### Best fit

- Consider as an abstraction target (protocol/interface shape), but **not** as near-term iOS delivery primitive.

---

## 3.3 Android option: Foreground service + bound service interfaces

### Summary

Run runtime ownership in a foreground-capable service where necessary and expose bound IPC interfaces to app surfaces.

### Pros

- Better support for longer-running background/continuity tasks than strict activity-bound execution.
- Natural fit for central runtime arbitration on Android.
- Compatible with explicit lifecycle notifications and service-level observability.

### Cons

- User-visible foreground service UX/notification burden.
- Newer Android versions impose stricter start/background limits.
- Requires explicit policy for battery, restart, and process reclaim scenarios.

### Best fit

- Runtime host/broker on Android when cross-app or long-running continuity is required.

---

## 3.4 Shared container approach (cross-platform storage bus)

### Summary

Use a shared storage/container queue as the canonical cross-process handoff substrate; pair with per-platform signaling mechanisms.

### Pros

- Most portable option across iOS and Android.
- Durable by default (survives process death and app switches).
- Simplifies replay/recovery/idempotency with event-log semantics.

### Cons

- Not real-time by itself; requires signaling/wakeup mechanisms.
- Requires strict schema evolution/versioning discipline.
- Potential contention/locking issues without careful write ownership rules.

### Best fit

- Canonical message envelope store, request/response queue, lease tokens, approval intents/results, inference continuation snapshots.

---

## 4) Recommended strategy

## 4.1 Recommendation (preferred)

Adopt a **Hybrid Broker + Durable Queue** model:

1. **Canonical IPC contract = durable message bus in shared container** (cross-platform).
2. **Platform-specific transport/wakeup adapters**:
   - iOS: App Extension entry points + shared App Group queue.
   - Android: Foreground/bound service as broker + shared durable queue.
3. **Single logical Runtime Broker contract** in shared core (`packages/ai-runtime` + `packages/mesh` interfaces), regardless of platform.

This gives reliable at-least-once delivery semantics, deterministic recovery after app/process death, and migration flexibility without requiring immediate hard multi-process runtime hosting on iOS.

## 4.2 Why this over alternatives

- More realistic near-term than iOS XPC-heavy designs.
- More reliable than transient in-memory IPC alone.
- Keeps single-process mode viable while introducing process-safe boundaries incrementally.

---

## 5) Canonical IPC contract (v1)

All cross-app/process messages should use one envelope schema:

- `messageId` (UUID)
- `conversationId` (stable session correlation)
- `originAppId` / `targetAppId`
- `domainContext` (`mind|learn|connect|training` etc.)
- `messageType` (`inference.request`, `inference.result`, `approval.request`, `approval.result`, `approval.rehost.request`, `handoff.snapshot`, `cancel.request`, `lease.update`)
- `priority` (`interactive|background`)
- `deadlineMs` / `ttlMs`
- `idempotencyKey`
- `payloadRef` (shared container blob pointer)
- `createdAt`, `attemptCount`, `schemaVersion`

### Delivery semantics

- **At-least-once** delivery with idempotent handlers.
- **Single-writer lease per queue partition** to prevent conflicting consumers.
- **Poison/dead-letter lane** after bounded retries.
- **Strict schema versioning** with backward-compatible readers.

---

## 6) Open question resolutions

## 6.1 When (if ever) domains become separate apps

Use explicit triggers instead of pre-committing to a split date.

### Split triggers

- Regulatory or policy isolation requiring separate binaries.
- Distribution/monetization needs per domain.
- Team velocity/deployment decoupling blocked by monolith release cadence.
- Runtime stability gains from process isolation outweighing UX costs.

### Recommendation

- Keep a **single app through Phase A/B** while implementing IPC seams.
- Reassess split readiness once metrics show sustained value from isolation and operations maturity.

---

## 6.2 In-flight inference during app switch

### Policy decision

Adopt **Queue + Resume Token** as default behavior, with explicit cancellation path.

### Rules

1. On app switch, active inference transitions to `handoff.pending`.
2. Runtime broker persists continuation snapshot + resume token.
3. Target app may:
   - `resume` if TTL and capability checks pass,
   - `cancel` explicitly,
   - or `expire` on timeout.
4. If target app does not attach before `handoffTimeoutMs`, inference is canceled with reason `handoff_timeout`.

### Deterministic outcomes

- No silent transfer.
- No duplicate finalization.
- User-visible status always one of: `resumed`, `canceled_by_user`, `canceled_timeout`, `failed_recovery`.

---

## 6.3 Delegator approval UI ownership in cross-app scenario

### Policy decision

Use **Originating App Owns Approval UI** with broker-mediated fallback.

### Rules

1. Approval requests are emitted to shared queue with `originAppId`.
2. Origin app is primary renderer for approval UX.
3. If origin app unavailable within approval SLA, broker can issue `approval.rehost.request` to eligible fallback app.
4. Approval resolution always records:
   - actor app,
   - actor identity/session,
   - delegated action id,
   - monotonic decision timestamp.

### Rationale

- Keeps user trust model stable (approval shown where action originated).
- Still supports continuity when app/process is unavailable.

---

## 7) Migration path from single-process

## Phase 0 — Baseline hardening (now)

- Introduce runtime broker interface **inside current process**.
- Normalize request/result/approval envelope types.
- Add idempotency keys and lifecycle states for inference and approvals.

## Phase 1 — Durable queue in single app

- Implement shared-queue semantics locally first (same process).
- Validate retry, dedup, dead-letter, and timeout behavior.
- Add observability counters and trace correlation IDs.

## Phase 2 — Cross-process within same app family

- iOS: wire App Group storage + extension ingress.
- Android: wire service broker + bind/notify flow.
- Keep business contracts unchanged; only transport adapters differ.

## Phase 3 — Optional multi-app split

- Promote selected domain to separate app bundle.
- Reuse broker contracts and queue semantics.
- Enable Delegator approval rehost fallback and cross-app lease arbitration.

## Phase 4 — Optimization

- Priority queue tuning by domain/intent.
- Prewarm strategies for expected handoff paths.
- Adaptive timeout policies from production telemetry.

---

## 8) Risks and mitigations

1. **Risk: queue duplication / race conditions**
   - Mitigation: strict idempotency keys + single-writer leases + monotonic state transitions.

2. **Risk: platform lifecycle interruptions**
   - Mitigation: durable snapshots, resumable requests, bounded TTLs.

3. **Risk: approval spoofing or stale approvals**
   - Mitigation: signed approval envelopes + app/session binding + expiry windows.

4. **Risk: user confusion during app switch**
   - Mitigation: explicit status model and standardized transition copy in all surfaces.

---

## 9) Decision summary

- Preferred strategy: **Hybrid Broker + Durable Queue (shared container first)**.
- iOS near-term: **App Extension ingress + App Group queue**, avoid assuming general XPC-service hosting model for shipped iOS multi-app runtime.
- Android near-term: **Foreground/bound service broker + durable queue**.
- In-flight inference: **Queue + Resume Token**, deterministic timeout/cancel semantics.
- Delegator approvals: **Origin app owns UI**, with broker-mediated fallback rehost.
- Migration: phased, contract-first, single-process compatible from day one.