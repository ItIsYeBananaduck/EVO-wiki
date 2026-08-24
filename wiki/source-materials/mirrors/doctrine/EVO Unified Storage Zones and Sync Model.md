---
title: EVO Unified Storage Zones and Sync Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVO Unified Storage Zones and Sync Model.md"]
updated: 2026-07-24
---

# EVO Unified Storage Zones and Sync Model
## Purpose

Define how data is stored, synchronized, and organized across all EVO applications.

This model ensures:

- strict on-device execution
- zero company-owned user data
- safe multi-device continuity
- scalable storage across low and high capacity devices
- consistent behavior across all EVO apps

---

## Core Principle

All cognition happens on-device.

The cloud is:

- user-owned
- optional
- used for continuity and storage expansion
- never the authoritative system of record owned by EVO

---

## Storage Model Overview

EVO uses a three-zone storage architecture:

### Zone 1 — Local Working State (Hot Layer)

### Zone 2 — Hidden Cloud System State (System Layer)

### Zone 3 — Visible EVO Workspace (User Layer)

---

## Zone 1 — Local Working State

### Definition

The on-device canonical working environment.

### Characteristics

- primary execution layer
- Alice operates here exclusively
- fully functional offline
- authoritative for active edits

### Contains

- active notes (scratch, comments, live notes)
- active tasks and project data
- working memory artifacts
- hydrated cloud data (temporary)

### Rules

- always exists
- always writable
- never depends on network
- may be partially evicted under storage pressure

---

## Zone 2 — Hidden Cloud System State

### Definition

User-owned cloud storage for system-managed artifacts.

### Characteristics

- not intended for user browsing
- implementation-facing
- may change structure over time
- supports sync, recovery, and system continuity

### Contains

- logs
- adapter copies (LoRAs)
- sync journals
- internal metadata
- cache snapshots
- transport artifacts

### Rules

- always user-owned (iCloud, Drive, etc.)
- never processed on EVO servers
- may be regenerated locally if lost (when possible)

---

## Zone 3 — Visible EVO Workspace

### Definition

User-facing, browsable cloud storage.

### Characteristics

- structured and human-readable
- stable organization
- represents user-owned artifacts
- accessible outside EVO apps

### Example Structure

```plain text
EVO/
  Connect/
    Projects/
    Notes/
    Archives/
  Training/
    Plans/
    Progress/
  Mind/
    Journals/
    Echo/
  Learn/
    Lessons/
    Progress/
  System/
    Hidden/
```

### Contains

- live notes (exported or materialized)
- comments (optional visibility)
- project artifacts
- archived user knowledge
- exports

### Rules

- must remain stable over time
- must be understandable by the user
- should not expose internal implementation details

---

## Storage Authority Model

Users can choose where active data lives.

### Modes

- local device is authority
- cloud mirrors data
- best for performance and privacy

- selected cloud acts as authority
- local device hydrates working copies
- ideal for low-storage devices

---

## Execution Rule (Critical)

Alice always executes locally.

Even in cloud-first mode:

- data is hydrated locally
- all edits occur locally
- changes sync back to the cloud

Cloud never becomes an execution environment.

---

## Data States

Each object may exist in one of the following local states:

- `not_local` → exists only in cloud
- `hydrated` → temporarily available locally
- `pinned` → forced to remain locally

---

## Sync States

- `clean` → no pending changes
- `dirty_local` → local changes not yet synced
- `syncing` → in progress
- `offline_pending` → waiting for connectivity
- `conflict` → requires resolution

---

## Hydration Model

### Hydration Triggers

- user opens an object
- Alice needs the object
- dependency resolution
- prefetch (optional)

### Behavior

- object is pulled from cloud to local
- becomes editable immediately
- marked for sync after modification

---

## Eviction Model

### Purpose

Prevent local storage exhaustion.

### Eviction Candidates

- old notes
- archived content
- inactive projects
- non-pinned hydrated data

### Rules

- never evict pinned data
- never evict unsynced changes
- cloud must have a clean copy before eviction

---

## Sync Model

### Core Behavior

- local changes are written first
- sync engine propagates changes to cloud
- cloud copies propagate to other devices

### Supported Paths

- local → cloud
- cloud → local
- cloud → cloud (via replication logic)

---

## Replication Model

Users may define:

### Authority Location

- local
- iCloud
- Google Drive
- Dropbox
- other providers

### Replication Targets

- one or more additional clouds

This creates a user-controlled storage graph.

---

## Artifact Classification

Each artifact must be categorized:

### Hidden System

- stored in Zone 2
- not user-facing

### User Visible

- stored in Zone 3
- browsable and meaningful

### Hybrid

- internal canonical form (Zone 1)
- exported/materialized version (Zone 3)

---

## Pins, Scratch Sync, and Offline Availability

These are separate concepts and must not be conflated.

### Pins

Pins control UI visibility.

A pinned item remains easy to reach in the interface.

Pinning does not change note type.

A pinned Scratch is still a Scratch.

---

### Scratch Sync Policy

Scratch sync behavior is controlled by user policy.

- unpinned scratches are local-only
- pinned scratches are syncable

- sync no scratches
- sync pinned scratches only
- sync all scratches

This changes continuity behavior only.

It does not make scratches auto-updatable or project-bound.

---

### Offline Availability

Offline availability controls local residency.

This should exist at the structural level:

- Project
- Phase
- Task
- Subtask

When a branch is marked Available Offline:

- that branch remains hydrated locally
- descendant structure remains hydrated locally
- attached Comments remain hydrated locally
- attached Live Notes remain hydrated locally
- local eviction is blocked for that branch while the setting remains enabled

Pins do not control offline residency.

Scratch pins affect visibility.

Offline toggles affect storage residency.

---

## Conflict Philosophy

- local changes are never silently discarded
- conflicting versions may be preserved
- resolution may be:
    - user-driven
    - assisted by Alice
- no centralized server arbitration

---

## Multi-Device Behavior

- device A writes locally → syncs to cloud
- device B hydrates from cloud → continues work
- devices remain loosely consistent through user cloud

---

## Offline Behavior

- system must work fully offline
- all operations happen locally
- sync occurs when connectivity returns

---

## Principle

EVO does not store user cognition.

EVO enables users to store, move, and extend their cognition across devices using their own storage systems.

The device performs the work.

The cloud carries the memory.

The user owns both.

## Related
