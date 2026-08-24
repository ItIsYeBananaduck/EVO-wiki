---
title: EVO On-Device First Principle
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVO On-Device First Principle.md
updated: 2026-07-24
---

# EVO On-Device First Principle
## Core Principle

EVO is an on-device-first platform. All user data is owned by the user and lives on their device. EVO does not operate a platform-controlled backend for user data storage.

This is not a sync strategy. It is a product constraint.

---

## What this means

### On-device is primary

- All EVO data (Methods, Talents, Tasks, Executions, Plans, Vault, etc.) is written to the local device store first
- The app is fully functional with no network connection
- Nothing requires a connection to an EVO-operated server to function

### User cloud is optional and user-chosen

- The only permitted cloud layer is the user's preferred cloud provider
- Supported providers: iCloud (CloudKit / iCloud Drive), Google Drive, Dropbox
- The user configures their provider; EVO does not choose for them
- The user can defer cloud setup entirely and remain fully on-device
- The user can change providers; data migrates to the new provider

### No platform-controlled backend for user data

- EVO does not store user data in a backend it operates (e.g. Supabase, [Fly.io](http://fly.io/), Firebase)
- No EVO-controlled server sits in the sync path
- Any implementation that routes user data through a platform-owned backend violates this principle

---

## What this does not restrict

- **Signaling / coordination** — lightweight, stateless coordination (e.g. Cloudflare signaling for Hive P2P) is permitted as long as user data does not transit or persist there
- **Auth / identity** — authentication flows may use external services; this principle governs data storage, not identity
- **EVO-to-EVO sharing** — user-initiated sharing between EVO users is a separate concern and must be designed to respect user ownership
- **Real-time collaboration** — future consideration; any implementation must not require a platform-owned backend for user data

---

## Enforcement rules

1. New persistence issues must target on-device local storage, not a platform backend
2. Any new issue that proposes writing user data to a platform-controlled store (Supabase, etc.) must be flagged and re-scoped
3. The `CloudSyncAdapter` interface (see EVOS1-258) is the approved abstraction for cloud sync — implement against it, not against a specific provider directly
4. The Supabase removal effort is tracked under EVOS1-199 — new work must not regress against it

---

## Known violations being corrected

| Issue | Description | Status |
| --- | --- | --- |
| EVOS1-199 | Re-architect Hive as local-first P2P — remove Supabase control plane | Todo |
| EVOS1-231 | Supabase migrations for Methods — paused, blocked on correct architecture | Todo |

---

## Related architecture work

- **EVOS1-258** — Define on-device-first persistence architecture: local store + `CloudSyncAdapter`
- **EVOS1-199** — Re-architect Hive as local-first P2P system
- **EVOC-244** — Approved Method Library (pending correct persistence target)

---

## Final Principle

The user's data belongs to the user. It lives where they can see it, control it, and take it with them. EVO is a tool the user runs — not a service that holds their data.
^[source-materials/mirrors/doctrine/EVO On-Device First Principle.md]
