---
title: "EVOC-252 — HiveStateSyncService production wiring decision"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-08-19
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-252 — HiveStateSyncService production wiring decision

## Decision

**WAN mailbox sync is always-on in production** (not feature-flagged).

> **Security prerequisite — EVOC-246 required before enabling WAN.**
> `HiveRuntimeWiring.enableWanMailboxSync = true` MUST NOT be activated in
> production until EVOC-246 (ciphertext envelope protection for Hive mailbox
> payloads) has been fully deployed and validated. Enabling WAN transport
> without EVOC-246 exposes raw state-sync payloads (leases, capabilities,
> work-tickets, messages) in transit. See the [Rollout guard plan](#rollout-guard-plan)
> section below.

## Rationale

- `HiveStateSyncService` is the common transport surface for lease, capability,
  work-ticket, and message propagation.
- Gating WAN transport behind a runtime flag would create split-brain behavior
  between LAN-only and LAN+WAN peers within the same hive.
- EVO-8 requires reliable cross-device mailbox sync behavior; production wiring
  should be deterministic across signed-in sessions.

## Security prerequisite: EVOC-246

Always-on WAN (`HiveRuntimeWiring.enableWanMailboxSync = true`) depends on
**EVOC-246 — ciphertext envelope protection** being active. EVOC-246 wraps every
`HiveStateEvent` payload in an encrypted envelope before it leaves the device
and decrypts it on receipt, ensuring no plaintext state is exposed through the
Cloudflare Worker mailbox.

Do not merge a build with `enableWanMailboxSync = true` into a production release
channel until EVOC-246 has passed security review and integration validation.

## Rollout guard plan

If EVOC-246 is not yet active when this flag is ready to ship, apply the
following guards:

1. **Feature-flag gating**: Keep `HiveRuntimeWiring.enableWanMailboxSync = false`
   in the production build until EVOC-246 is confirmed active on all supported
   platforms. Use a remote config entry (e.g., Supabase remote config or a
   server-side flag) to flip it without a binary release.

2. **Phased rollout**: Enable WAN sync for an internal dogfood cohort first,
   then a 5 % canary, then full production. Gate each phase on zero new security
   incidents and no ciphertext validation errors in logs.

3. **Monitoring criteria**: Track `[HiveMailboxSync] decrypt error` log lines and
   alert if the rate exceeds 0.1 % of events in any 5-minute window. Treat any
   decryption failure as a potential envelope-integrity incident.

4. **Kill-switch criteria**: Immediately set `enableWanMailboxSync = false` via
   remote config if: (a) EVOC-246 envelope validation fails on > 0.1 % of
   events, (b) any plaintext payload is detected in the Cloudflare Worker inbox,
   or (c) a security incident is raised against the mailbox transport layer.

## Implementation standard

- `HiveRuntimeWiring.enableWanMailboxSync` is the single wiring decision point.
- Production value is `true` **only after EVOC-246 is deployed and validated**
  (see [Security prerequisite: EVOC-246](#security-prerequisite-evoc-246)).
- `HiveStateSyncService` wires mailbox start/stop/broadcast from that policy;
  the EVOC-246 envelope layer must be active whenever this flag is `true`.
- `HomeScreen` always initializes Hive services for signed-in users and relies
  on the central wiring policy for WAN enablement; it inherits the EVOC-246
  prerequisite by extension.

## Files aligned

- `flutter_app/lib/core/hive/hive_runtime_wiring.dart` — wiring decision point;
  set `enableWanMailboxSync = true` only after EVOC-246 is active.
- `flutter_app/lib/core/hive/hive_state_sync.dart` — consults wiring policy for
  WAN start/stop/broadcast; relies on EVOC-246 envelope layer being in place.
- `flutter_app/lib/features/home/presentation/home_screen.dart` — initializes
  Hive services for signed-in users; WAN enablement follows central policy and
  the EVOC-246 prerequisite.