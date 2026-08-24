---
title: MOC Protected Storage and Access Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/MOC Protected Storage and Access Model.md"]
updated: 2026-07-24
---

# MOC Protected Storage and Access Model
## Purpose

Define the protected storage, classification, inheritance, and controlled access model for EVO systems.

This note cluster formalizes how EVO distinguishes between:

- normal AI-visible working space
- protected storage containers
- protected runtime artifacts
- controlled temporary access
- repo-level protected paths
- metadata-governed protected system objects

This architecture exists to ensure that users can safely decide what Alice may access, what she may never access, and what may only be accessed through explicit approval and authenticated temporary sessions.

---

## Core Principle

EVO does not only protect the AI from the outside world.

EVO also protects user-selected data, files, projects, and system objects from Alice.

This is achieved through:

- path-based protected zones for repo and raw filesystem enforcement
- metadata-based protection classification for system-owned objects
- Delegator-enforced denial and exception handling
- temporary scoped access sessions instead of permanent unprotection
- privileged loading for protected runtime artifacts that must shape Alice without becoming normal Alice-readable content

---

## Key Concepts

### Normal Workspace

The ordinary space where Alice can read, reason, organize, and act within her allowed permissions.

### Protected Zone

Any area, object, or artifact that Alice cannot access through ordinary execution paths.

### Bunker

A user-facing protected storage container that may exist locally, in cloud storage, or as a Connect-managed protected object space.

### Protected Runtime Artifact

A runtime-consumed artifact that influences Alice or the system but should not be readable by Alice as ordinary workspace knowledge.

### Delegator

The enforcement layer that blocks, allows, or conditionally grants access to protected targets.

### Access Session

A temporary, scoped exception that allows specific protected targets to be accessed for a limited time and purpose without permanently removing their protection.

## System Layers

- User Layer → Bunkers, Onboarding
- Enforcement Layer → Delegator, Path Rules
- Data Layer → Metadata, Inheritance
- Runtime Layer → Privileged Loader

---

## Model Summary

### Repo and raw filesystem

Protection is enforced using path and naming conventions.

Example:

- `.evo_env/**`

### System-owned objects

Protection is enforced using metadata classification.

Examples:

- projects
- phases
- tasks
- subtasks
- notes
- artifacts
- managed protected folders

### Enforcement

Delegator does not rely directly on user-facing labels.

Delegator enforces:

- resolved protected paths
- resolved protected object targets
- scoped access session exceptions

---

## Recommended Supporting Notes

- [Protection Classification and Inheritance Model](https://www.notion.so/343c72bad01381928bd0fb9c94164e20)
- [Bunker Model — Protected User and System Storage Containers](https://www.notion.so/343c72bad0138186af70ec9b2ce2ba9f)
- [Bunker Access Session Model](https://www.notion.so/343c72bad01381788004c1ec4b0b695d)
- [Protected Runtime Artifact and Privileged Loader Model](https://www.notion.so/343c72bad0138159ae9ed38ce9f429db)
- [Bunker Onboarding and First-Run Trust Model](https://www.notion.so/343c72bad01381319a8dddf90c1f2829)

---

## Status

Foundational architecture.

This cluster should be considered canonical for future Bunker and protected storage work.

---

## Related Notes

- [Protection Classification and Inheritance Model](https://www.notion.so/343c72bad01381928bd0fb9c94164e20)
- [Bunker Model — Protected User and System Storage Containers](https://www.notion.so/343c72bad0138186af70ec9b2ce2ba9f)
- [Bunker Access Session Model](https://www.notion.so/343c72bad01381788004c1ec4b0b695d)
- [Protected Runtime Artifact and Privileged Loader Model](https://www.notion.so/343c72bad0138159ae9ed38ce9f429db)
- [Bunker Onboarding and First-Run Trust Model](https://www.notion.so/343c72bad01381319a8dddf90c1f2829)
