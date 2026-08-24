---
title: EVO Sync Core Package Spec
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVO Sync Core Package Spec.md
updated: 2026-07-24
---

# EVO Sync Core Package Spec
## Purpose

Define the shared sync and storage orchestration package used across all EVO applications.

This package exists to ensure:

- one consistent data sovereignty model across EVO
- on-device execution in every app
- user-owned cloud continuity without company-owned cognition storage
- shared sync, hydration, eviction, and replication behavior
- app-specific sync policies built on one common framework

---

## Core Principle

EVO apps must not each invent their own sync philosophy.

They may sync different artifact types, but they must all obey the same storage doctrine.

The sync core defines:

- how sync works
- how local execution works
- how hydration works
- how eviction works
- how user-owned cloud replication works

Each app defines:

- what artifacts participate
- which artifacts are hidden
- which artifacts are visible
- what policy applies to each artifact

---

## Package Role

The shared sync package is responsible for:

- storage authority mode handling
- sync state tracking
- hydration and eviction orchestration
- transport abstraction
- replication routing
- conflict handling hooks
- sync journaling
- app adapter integration

It is not responsible for:

- app-specific artifact meaning
- UI policy
- domain-specific merge semantics
- note or task business logic

---

## Recommended Package Structure

```plain text
packages/evo_sync_core/
  lib/
    src/
      engine/
      transport/
      hydration/
      eviction/
      journaling/
      policies/
      adapters/
```

---

## Core Concepts

### Storage Authority Mode

Determines where active data lives.

Supported modes:

- `local_first`
- `cloud_first`

### Local State

Determines local presence of an object:

- `not_local`
- `hydrated`
- `pinned`

### Sync State

Determines current sync condition:

- `clean`
- `dirty_local`
- `syncing`
- `offline_pending`
- `conflict`

### Artifact Class

Determines routing:

- `hidden_system`
- `user_visible`
- `hybrid`

---

## Main Interfaces

### SyncEngine

Owns the overall sync lifecycle.

Responsibilities:

- initialize sync system
- track enabled state
- trigger sync runs
- emit sync state
- coordinate app adapters
- invoke transport layer

---

### SyncTransport

Abstracts the cloud provider.

Examples:

- iCloud
- Google Drive
- Dropbox
- future custom providers

Responsibilities:

- read file
- write file
- delete file
- list files
- check existence
- get modified time
- trigger provider sync if supported

---

### HydrationManager

Responsible for materializing cloud-resident data locally.

Responsibilities:

- hydrate requested object
- prefetch if needed
- mark local state
- provide local working copy
- ensure Alice always works on a local copy

---

### EvictionManager

Responsible for reducing local storage pressure safely.

Responsibilities:

- identify eviction candidates
- never evict dirty data
- never evict pinned data
- confirm cloud copy is safe before eviction
- downgrade hydrated data back to cloud-resident state

---

### SyncJournal

Tracks changes and sync intent.

Responsibilities:

- record local mutations
- track pending sync items
- support retry after offline periods
- support future conflict review

---

### AppSyncAdapter

Each EVO app implements this to describe its own syncable artifact domains.

Responsibilities:

- declare artifact categories
- enumerate local sources
- determine visibility class
- determine sync eligibility
- declare merge strategy hooks
- expose hydration rules if app-specific

---

## App Adapter Examples

### Training Adapter

Would define artifacts like:

- conversation logs
- preferences
- workout history
- nutrition logs
- LoRA adapter copies

This mirrors what Training already syncs today through its manager and iCloud transport layer.

### Connect Adapter

Would define artifacts like:

- comments
- live notes
- tasks
- project structures
- archived project artifacts

### Mind Adapter

Would define artifacts like:

- journals
- Echo artifacts
- reflection archives

### Learn Adapter

Would define artifacts like:

- lesson state
- curriculum progress
- child-safe learning artifacts

---

## Storage Zones Integration

The sync core must work with the unified storage zone model.

### Zone 1 — Local Working State

Used for:

- active execution
- active edits
- hydrated working copies

### Zone 2 — Hidden Cloud System State

Used for:

- logs
- adapters
- internal metadata
- sync journals
- machine-facing artifacts

### Zone 3 — Visible EVO Workspace

Used for:

- exported live notes
- visible user artifacts
- archives
- project-facing material

The sync core does not decide which zone an artifact belongs to.

Each app adapter declares that.

---

## Execution Rule

Alice always executes locally.

Even in cloud-first mode:

- object is hydrated locally
- edits happen locally
- sync core pushes changes back outward

The sync core must never require remote execution for active cognition.

---

## Hydration Rules

### Triggers

- user opens an object
- Alice needs an object
- object is marked available offline
- prefetch policy requests it

### Requirements

- hydration must create a local working copy
- object becomes editable immediately
- hydration state must be recorded

---

## Eviction Rules

### Requirements

- never evict dirty local data
- never evict pinned data
- never evict available-offline branches
- only evict after safe cloud replication
- preserve enough metadata to rehydrate later

---

## Scratch Policy Support

The sync core must support scratch continuity policy without confusing it with UI pinning.

### Scratch Sync Policies

- `none`
- `pinned_only`
- `all`

### Important distinction

Pins control UI visibility.

Scratch sync policy controls sync eligibility.

The sync core should consume resolved policy, not own UI pin semantics itself.

---

## Offline Availability Support

The sync core must support structural offline residency.

Supported levels:

- Project
- Phase
- Task
- Subtask

When marked available offline:

- branch remains hydrated locally
- descendant artifacts remain hydrated locally
- eviction is blocked for that branch

---

## Replication Model

Users may choose:

- authority location
- replication targets

Examples:

- local authority + iCloud mirror
- local authority + Drive + Dropbox mirror
- cloud-first authority on iCloud + secondary Drive replication

The sync core should support one authority source plus optional additional replication targets.

---

## Conflict Philosophy

The sync core must preserve user cognition over convenience.

Rules:

- never silently discard local changes
- allow assisted conflict resolution later
- preserve conflicting versions when needed
- avoid centralized server arbitration

---

## Migration Path from Training

Training currently contains a first-generation sync implementation with:

- a sync manager
- a transport abstraction
- category-based sync routines
- local file-first behavior with cloud transport

The sync core package should extract and generalize:

- SyncEngine shape
- transport abstraction
- category-based orchestration
- provider bridge pattern

Training then becomes:

- first app adapter implementation

Connect becomes:

- second app adapter implementation

---

## MVP Package Deliverables

### Phase 1

- shared `SyncEngine`
- shared `SyncTransport`
- shared sync state model
- shared local state model
- first Training adapter extraction

### Phase 2

- hydration manager
- eviction manager
- sync journal
- Connect adapter integration

### Phase 3

- multi-provider replication
- conflict review hooks
- archive/offload support
- visible workspace materialization helpers

---

## Principle

EVO Sync Core is not a cloud backend.

It is the local orchestration layer that allows EVO apps to move user-owned cognition safely across devices and storage locations without centralizing trust in the company.

## Related

^[source-materials/mirrors/doctrine/EVO Sync Core Package Spec.md]
