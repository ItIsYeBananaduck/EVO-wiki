---
title: IPC Strategy — Multi-App Domain Separation
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/IPC Strategy — Multi-App Domain Separation.md
updated: 2026-07-24
---

# IPC Strategy — Multi-App Domain Separation

> NOTE: This is a canonical doctrine note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

---

## Purpose

Define how EVO communicates across app processes if domains (`mind`, `learn`, `connect`, `training`) are split into separate app bundles. Today the system is single-process; this spec introduces the seams required for future process/app boundaries without forcing a split.

---

## Core Principle

The IPC contract is **durable-queue-first and migration-safe**. Delegator approval governance and tool authority must remain deterministic across process boundaries. The single-process runtime remains valid; only envelope and queue semantics are introduced now.

---

## Definitions

- **Runtime Broker** — the logical arbiter of inference requests and cross-app message routing
- **Shared container** — the canonical cross-process handoff substrate (App Group on iOS, durable service queue on Android)
- **Handoff token** — a durable resume snapshot persisted when an app switch interrupts in-flight inference
- **Approval rehost** — broker-mediated fallback that transfers approval UI ownership when the originating app is unavailable

---

## System Structure

### Recommended Strategy: Hybrid Broker + Durable Queue

1. **Canonical IPC contract** = durable message bus in shared container (cross-platform)
2. **Platform-specific transport adapters**:
   - iOS: App Extension ingress + shared App Group queue
   - Android: Foreground/bound service as broker + shared durable queue
3. **Single logical Runtime Broker contract** defined in `packages/ai-runtime` + `packages/mesh` interfaces

### Message Envelope Schema (v1)

| Field | Description |
|---|---|
| `messageId` | UUID |
| `conversationId` | stable session correlation |
| `originAppId` / `targetAppId` | app bundle identifiers |
| `domainContext` | `mind\|learn\|connect\|training` |
| `messageType` | `inference.request`, `inference.result`, `approval.request`, `approval.result`, `handoff.snapshot`, `cancel.request`, `lease.update` |
| `priority` | `interactive\|background` |
| `deadlineMs` / `ttlMs` | delivery window |
| `idempotencyKey` | dedup key |
| `payloadRef` | shared container blob pointer |

### Delivery Semantics

- At-least-once delivery with idempotent handlers
- Single-writer lease per queue partition (no conflicting consumers)
- Poison / dead-letter lane after bounded retries
- Strict schema versioning with backward-compatible readers

---

## Rules

1. **Safety > convenience** — Delegator approval and tool governance remain deterministic across process boundaries
2. **No hard dependency on continuous background execution** — IPC model must tolerate suspension, process death, and delayed delivery
3. **Cross-platform parity at contract level** — iOS and Android implementations may differ internally; message contracts and orchestration semantics stay shared
4. **Origin app owns approval UI** — broker may rehost only if origin app is unavailable within approval SLA; resolution always records actor app, identity, action id, and decision timestamp

---

## Flow

### In-flight inference during app switch

1. On app switch, active inference transitions to `handoff.pending`
2. Runtime broker persists continuation snapshot + resume token
3. Target app may `resume` (if TTL and capability checks pass), `cancel` explicitly, or `expire` on timeout
4. No silent transfer; no duplicate finalization
5. User-visible status: one of `resumed`, `canceled_by_user`, `canceled_timeout`, `failed_recovery`

### Migration phases

| Phase | Scope |
|---|---|
| 0 — Baseline | Runtime broker interface inside current single process; normalize envelope types |
| 1 — Durable queue | Shared-queue semantics locally; validate retry, dedup, dead-letter, timeout |
| 2 — Cross-process | iOS: App Group + extension ingress; Android: service broker + bind/notify |
| 3 — Multi-app split | Promote domain to separate app bundle; reuse broker contracts |
| 4 — Optimization | Priority queue tuning, prewarm strategies, adaptive timeout policies |

---

## Relationships

See also: [[Domain Authority map]], [[Delegator — Execution Governance Doctrine]], [[Always-On Alice Hosting Models]]

---

## Edge Cases / Special Handling

- **iOS XPC**: clean interface but materially constrained for App Store distributed multi-app runtime on iOS — not a near-term delivery primitive; consider as abstraction target only
- **Android foreground service**: required for longer-running continuity tasks; new Android versions impose stricter start/background limits — policy must cover battery, restart, and process reclaim
- **Queue contention**: use strict single-writer leases; event-log semantics simplify replay and recovery

---

## Summary

EVO's IPC strategy uses a Hybrid Broker + Durable Queue model. The canonical cross-process contract is a durable message bus in a shared container, with platform-specific transport adapters. Today's single-process runtime is unchanged; this spec introduces envelope semantics and phased migration seams. Delegator approvals stay origin-app-owned with broker-mediated rehost fallback.

## Related

^[source-materials/mirrors/doctrine/IPC Strategy — Multi-App Domain Separation.md]
