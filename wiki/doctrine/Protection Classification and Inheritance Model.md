---
title: Protection Classification and Inheritance Model
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Protection Classification and Inheritance Model.md
updated: 2026-07-24
---

# Protection Classification and Inheritance Model
## Purpose

Define how protection is classified across EVO systems and how protected state inherits across system-owned objects.

This note exists to formalize the distinction between:

- path-based protection for repo and raw filesystem structures
- metadata-based protection for system-owned objects

---

## Core Principle

Protection source-of-truth differs by storage model.

### Repo and raw filesystem

Use path-based naming and registration.

### System-owned objects

Use metadata classification.

Delegator enforces the resolved result in both cases.

---

## Classification Model

### Path-based classification

Used for:

- raw repo paths
- raw filesystem folders
- repo-internal protected developer zones

Example:

- `.evo_env/**`

This is best when no durable object metadata layer exists.

### Metadata-based classification

Used for:

- projects
- phases
- tasks
- subtasks
- notes
- artifacts
- managed containers
- Connect-owned storage objects

Recommended metadata field:

```json
{
  "protectionClass": "inherit"
}
```

Allowed values:

- `inherit`
- `protected_enclave`

---

## Why `normal` should not be the default child override

If a child inside a protected parent can casually be changed to normal, the system creates a silent leak risk.

A user or system could:

- temporarily downgrade a child
- forget to restore protection
- unintentionally leave a hole in the trust boundary

For this reason, the default metadata model should not include casual de-escalation.

---

## Inheritance Rule

Protection should flow downward by default.

### If a parent is protected

Its children inherit protected status unless a separate high-friction exception model exists.

### If a parent is normal

Its children remain normal unless explicitly promoted to protected.

---

## Effective Resolution Rule

The stored metadata may contain the declared value.

The system should compute an effective value.

Example:

```json
{
  "protectionClass": "inherit",
  "effectiveProtectionClass": "protected_enclave"
}
```

This allows the system to preserve inheritance while making enforcement explicit.

---

## Recommended Rule

Closest explicit value wins, but only in the direction of equal or stronger protection.

This means:

- normal parent → child may become protected
- protected parent → child should remain protected by inheritance
- protected parent → child should not casually de-escalate to normal

---

## Delegator Rule

Delegator should not directly care whether protection came from:

- a path convention
- a metadata field

Delegator should care only about:

- resolved protected targets
- registered protected paths
- active scoped exceptions

---

## Summary

Protection is:

- path-based for raw repo and filesystem structures
- metadata-based for system-owned EVO objects

Inheritance is sticky and should protect children by default.

Protected parents should not casually produce unprotected children.

---

## Related Notes

- Protected Storage and Access Model
- [Bunker Model — Protected User and System Storage Containers](https://www.notion.so/343c72bad0138186af70ec9b2ce2ba9f)
- [Bunker Access Session Model](https://www.notion.so/343c72bad01381788004c1ec4b0b695d)
^[source-materials/mirrors/doctrine/Protection Classification and Inheritance Model.md]
