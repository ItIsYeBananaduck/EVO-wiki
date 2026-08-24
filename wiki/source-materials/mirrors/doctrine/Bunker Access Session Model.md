---
title: Bunker Access Session Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Bunker Access Session Model.md"]
updated: 2026-07-24
---

# Bunker Access Session Model
## Purpose

Define how Alice may be granted temporary access to protected Bunker content without permanently removing protection.

This note exists to replace the unsafe idea of “temporarily removing something from a Bunker” with a safer session-based exception model.

---

## Core Principle

Protected content should remain protected.

Temporary work should happen through scoped, expiring access sessions.

Do not unprotect the object itself.

Do not move it out and trust that it will be moved back later.

---

## Bunker Access Session Definition

A Bunker Access Session is a temporary, scoped permission grant that allows Alice or another authorized system actor to access specific protected targets for a limited purpose and duration.

Example session metadata:

```json
{
  "sessionId": "abc123",
  "allowedTargets": [
    "/bunker/project/file.ts"
  ],
  "allowedActions": ["read_file", "write_file"],
  "expiresAt": "timestamp",
  "accessReason": "user_approved_task"
}
```

---

## Scope Rules

A session should be constrained by:

- exact target path or object
- allowed action type
- expiration time
- requesting actor
- user approval state
- authentication state

---

## Important Rule

Access is not global.

A Bunker Access Session should not make the Bunker visible as a whole.

It should allow only the explicitly approved target.

For example:

- allow `/bunker/project/file.ts`
- still deny listing `/bunker/project/`
- still deny reading `/bunker/project/other.ts`

---

## Authentication Rule

Creating or approving a Bunker Access Session should require strong local authentication, such as:

- system password
- biometric unlock
- OS-native secure confirmation

This should be treated as a trust-boundary crossing event.

---

## Session Lifecycle

### 1. Protected by default

Bunker target remains inaccessible.

### 2. User requests or approves access

The system prepares a scoped session request.

### 3. User authenticates

OS-native trust mechanism confirms the request.

### 4. Delegator registers temporary exception

Only approved targets and actions are accessible.

### 5. Session expires

Access is revoked automatically.

### 6. Bunker remains protected

No permanent downgrade occurs.

---

## Resource Management Use Case

Alice may detect:

- memory pressure
- CPU pressure
- storage pressure

Alice may notice that protected resources are contributing to that pressure.

Alice may not directly inspect or manipulate those protected resources without a valid session.

Correct behavior:

- detect protected resource pressure
- ask the user for permission
- require auth
- receive scoped session
- save/close/reopen only the approved targets
- revoke access when done

This preserves both autonomy and trust.

---

## Summary

Bunker Access Sessions allow temporary work inside protected storage without permanently changing protection state.

They are scoped, authenticated, auditable, and automatically expiring.

---

## Related Notes

- [Bunker Model — Protected User and System Storage Containers](https://www.notion.so/343c72bad0138186af70ec9b2ce2ba9f)
- [Protection Classification and Inheritance Model](https://www.notion.so/343c72bad01381928bd0fb9c94164e20)
- [Bunker Onboarding and First-Run Trust Model](https://www.notion.so/343c72bad01381319a8dddf90c1f2829)

## Doctrine Update — 2026-05-01 — Bunker Visibility in Connect Library

Bunkers are visible to the user in Connect Library as protected zones.

This note’s prior “hidden” language refers to Alice’s operational visibility, not the user’s view.

Revised model:

- The user can see Bunker containers in Connect Library.
- Alice may know a Bunker exists as a protected zone.
- Alice cannot browse, search, summarize, inspect, organize, or modify Bunker contents through normal execution paths.
- Access requires a scoped, expiring, authenticated session.
- Bunker contents are excluded from Alice’s default operational index.

Canonical reference:
EVOconnect — Connect Library and Bunker Visibility Model
