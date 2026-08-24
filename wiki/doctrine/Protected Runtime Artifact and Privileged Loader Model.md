---
title: Protected Runtime Artifact and Privileged Loader Model
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Protected Runtime Artifact and Privileged Loader Model.md
updated: 2026-07-24
---

# Protected Runtime Artifact and Privileged Loader Model
## Purpose

Define how protected runtime artifacts may influence Alice and the system without becoming ordinary Alice-readable content.

---

## Core Principle

Delegator denial and Bunker protection block ordinary access.

A separate privileged loading path is still needed for protected artifacts that must be consumed by the runtime.

---

## Why this exists

Without a privileged loader, one of two bad outcomes happens:

### Outcome 1

Alice cannot access the artifact, but neither can the runtime.

### Outcome 2

The runtime consumes the protected artifact through the same ordinary path Alice would use, weakening the boundary.

The privileged loader solves this.

---

## Loader Role

The privileged loader is a controlled runtime-owned path that may:

- read protected source artifacts
- compile or transform them
- inject derived runtime state
- keep the protected source outside Alice’s normal readability surface

It should not become a general-purpose bypass channel.

---

## Relationship to repo protection

For repo/runtime protection:

- `.evo_env/**` blocks ordinary Alice/tool access
- privileged loader may still load specific approved runtime artifacts from protected locations

This is not a contradiction.

This is the intended split:

- Delegator blocks ordinary access
- privileged loader enables controlled runtime consumption

---

## Relationship to Bunkers

For system-managed protected objects:

- Bunkers protect ordinary access
- privileged loader may consume specifically designated protected runtime artifacts when architecture requires it

Not every Bunker needs a loader.

Only runtime-consumed protected artifacts need one.

---

## Recommended rule

Keep the privileged loader narrow.

It should be:

- explicit
- runtime-owned
- limited to designated artifact classes
- non-discoverable through ordinary Alice workspace access

---

## Summary

Protected runtime artifacts still require a privileged loader.

Delegator denial prevents ordinary access.

The privileged loader provides controlled runtime consumption without exposing protected source material to Alice.

---

## Related Notes

- Protected Storage and Access Model
- [Bunker Model — Protected User and System Storage Containers](https://www.notion.so/343c72bad0138186af70ec9b2ce2ba9f)
^[source-materials/mirrors/doctrine/Protected Runtime Artifact and Privileged Loader Model.md]
