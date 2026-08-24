---
title: EVOconnect — Connect Library and Bunker Visibility Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOconnect — Connect Library and Bunker Visibility Model.md"]
updated: 2026-07-24
---

# EVOconnect — Connect Library and Bunker Visibility Model
## Purpose

Define the unified file and object access layer for EVOconnect.

Connect Library is not a macOS Finder clone and should not be named EVOfinder. It is a unified file graph across devices, clouds, and connected app sources.

The goal is to let the user see what they have everywhere in one place while keeping Alice's operational access bounded, permissioned, and auditable.

The Connect Library is also intended to function as a unified access layer across the broader workspace ecosystem — not only files, but also repos, apps, workflows, panes, views, Spaces, notes, and tasks.

The user should not need to think about where something is stored. The Library surfaces what matters when it matters.

Connect is an orchestration layer over cognition and storage surfaces, not a data owner.

---

## Core Concept

Connect Library gives the user a unified view of files and file-like objects across:

- local devices running Connect
- iCloud Drive
- Google Drive
- future cloud providers
- app artifact folders
- task attachments
- generated documents
- user-created project folders
- Bunkers

Files and cognition artifacts remain in their original domain systems and storage locations.

Connect governs and maintains:

- metadata overlays
- local-first indexes
- object references
- access policies and enforcement decisions
- scoped access session bindings
- audit logs
- organization suggestions
- user-facing browse/search surfaces

Connect is not a data owner and does not centralize domain cognition. It is a governed access and orchestration layer that unifies addressing, policy, and execution routing across those systems, with user/privacy boundaries enforced at every protected object interaction.

---

## Unified Operational Access

The Connect Library should unify access to:
- files
- folders
- repos
- apps
- workflows
- views
- panes
- notes
- tasks
- snippets
- saved workflows

This allows the Library to function as:
- an operational retrieval layer
- a workflow discovery system
- a workspace coordination surface

rather than only a filesystem viewer.

The user should be able to navigate their entire operational environment through one coherent access layer.

---

## User View vs Alice View

Connect has two visibility layers.

### User View

The user can see the full Connect Library, including Bunkers.

Bunkers are visible protected zones. They should not disappear from the user's unified file view.

### Alice Operational View

Alice does not receive the same view as the user.

By default, Alice can only see and act on open or explicitly allowed objects.

Bunker contents are hidden from Alice unless the user grants a scoped, expiring access session.

Alice may know that a Bunker exists as a protected zone, but she may not inspect its contents or use those contents in cognition or execution without permission.

---

## Revised Bunker Doctrine

A Bunker is not invisible storage.

A Bunker is a visible protected zone.

A Bunker should be:

- visible to the user in Connect Library
- visible to Alice only as a protected zone by default
- restricted from Alice's ordinary browse, search, summarize, organize, and execution paths
- accessible only through scoped, expiring user-approved access sessions
- audited whenever access is requested, granted, used, denied, or revoked

---

## File Organizer Role

File Organizer is an ongoing App Talent built on top of Connect Library.

Its job is to keep the user's open files organized and easily accessible.

Alice should organize loose files, not restructure the user's existing folder system.

---

## File Organizer Rules

Alice may automatically or semi-automatically:

- classify loose or random files
- suggest destinations
- move loose files into existing folders when allowed
- create new folders inside approved open zones when allowed
- tag or label files when supported
- maintain metadata and search indexes
- surface duplicates or stale files
- propose cleanup actions

Alice must not automatically:

- move pre-existing folders
- rename pre-existing folders
- delete files
- delete folders
- read Bunker contents
- move files into or out of a Bunker
- restructure protected zones
- expose protected metadata in logs
- perform cross-cloud moves without approval

---

## Existing Folders as Anchors

Existing folders are treated as anchors.

Alice may use existing folders as destinations, but she cannot move those folders automatically.

This protects the user's mental model of their storage.

---

## Loose Files as Organizable Objects

Loose files are the primary objects Alice organizes.

A loose file may be:

- a download
- a screenshot
- an exported document
- an attachment
- a generated artifact
- a scanned document
- an unsorted cloud file

Alice's default job is to place these files into the most useful existing or newly approved folder.

---

## Access Levels

Connect Library objects should support access levels such as:

- hidden from Alice
- metadata only
- read only
- suggest changes
- edit with approval
- organize automatically within approved bounds
- never access

The user can see more than Alice can use.

---

## Open Zone vs Bunker Semantics

Open Zone and Bunker are Connect-owned protection semantics derived from Connect metadata, not from raw repo path conventions.

- **Bunker**: effectiveProtectionClass = `protected_enclave`.
- **Open Zone**: any object whose effectiveProtectionClass is not `protected_enclave`.

This keeps policy portable across local, cloud, and app sources and avoids coupling enforcement to path patterns.

---

## Access-Level Enforcement and UI Mapping

Access level is resolved from Connect metadata + effective protection class + active scoped session state.

| Access level | Enforcement in Alice Operational View | UI presentation in User View |
| --- | --- | --- |
| hidden | Object excluded from operational index and retrieval. | User may still see object in full library; marked unavailable to Alice. |
| metadata-only | Alice may resolve identifiers and safe metadata only; no content read or tool execution. | Visible with metadata badge; preview/content actions disabled for Alice. |
| read-only | Alice may read/reason over permitted content; no write/move/delete. | Visible as readable; edit actions gated. |
| suggest | Alice may propose mutations but cannot execute them. | Suggestions shown with explicit approve/reject controls. |
| edit with approval | Alice may execute only after per-action or per-scope user approval. | Pending-approval state and audit trail shown. |
| edit automatically for File Organizer (pre-approved) | Alice may perform bounded organizer edits in approved Open Zones only; no Bunker writes without scoped session + approval. | Clearly labeled automation zone with reversible audit history. |
| never access | Hard deny regardless of talent/method unless a stronger explicit governance override exists. | Visible as protected/blocked boundary. |

For Bunker objects, default floor policy is hidden or never access unless a valid scoped access session elevates permissions for exact targets and allowed actions.

---

## Scoped Access Sessions

Bunker access requires a scoped access session.

A scoped access session must define:

- target Connect object ID (never raw path addressing)
- exact-target scope (no parent/sibling/child expansion)
- allowed actions list (enforced at validation time)
- explicit expiration
- reason
- approving user
- actor
- approval/auth method consistent with Connect UX
- audit record
- result

Session mechanics are reused from the shared primitive (EVOS1-197); Connect-side work is object addressing + wiring only.

Deny by default: when `effectiveProtectionClass` is protected and no active valid session exists for that exact object/action, access is denied for both reasoning and execution paths.

---

## Minimal Integration Path (EVOC-259)

1. Resolve requested resource into a stable Connect object ID in the unified graph.
2. Resolve `effectiveProtectionClass` for that object.
3. If object is not protected, proceed through normal policy checks.
4. If object is protected, require approval/auth gating and request/reuse a scoped session bound to that object ID.
5. Validate session: exact target match, requested action in allowed-actions list, and not expired.
6. Permit only when validation passes; otherwise deny and emit audit event.

This keeps Connect aligned with doctrine: cognition/data remains in domain systems, while Connect governs access boundaries and orchestration.

---

## Audit Requirements

Every organization action should produce an audit event.

Audit events should include:

- object ID
- source
- original location
- proposed destination
- action type
- Method or Talent ID
- approval status
- result
- timestamp
- actor
- validation result

---

## Relationship to Spaces

Different Spaces may expose different Library priorities.

Examples:

Development Space:
- repos
- snippets
- terminals
- Git workflows
- project folders

Research Space:
- saved references
- clipped content
- browser captures
- PDFs
- notes

Life Space:
- reminders
- lightweight documents
- personal organization
- quick capture workflows

The Connect Library adapts contextually to the active operational environment.

---

## Dockable Folders

Folders should be dockable to the Action Bar.

This is important because folders often represent:
- projects
- workflows
- operational environments
- active work contexts

Examples:
- active repo
- screenshots folder
- documentation folder
- design assets
- exports folder
- research directory

Docked folders allow users to quickly reopen operational contexts without manually navigating the filesystem repeatedly.

---

## Dockable Repositories

Repositories should behave like operational workspace objects.

A docked repo may expose:
- quick open
- Git state
- recent files
- related tasks
- related notes
- terminal shortcuts
- workflow restoration

This allows development workflows to feel tightly integrated into the larger workspace environment.

---

## Temporary File Panes

The Library may launch temporary file panes.

Examples:
- temporary folder explorer
- temporary repo browser
- temporary file preview
- temporary asset browser

Temporary panes should:
- pop out quickly
- avoid permanently occupying workspace space
- collapse back into the Action Bar when finished

This keeps the workspace flexible and lightweight.

---

## Workflow-Aware Retrieval

The Connect Library should eventually become workflow-aware.

Examples:
- recently active project folders
- currently relevant repos
- related tasks
- related notes
- related workflows
- active operational context

The goal is to reduce:
- repetitive navigation
- repeated searching
- operational fragmentation

Alice should help surface relevant workspace context automatically.

---

## Action Bar Integration

The Action Bar integrates tightly with the Connect Library.

Examples:
- docked folders
- docked repos
- quick-access workflows
- temporary file panes
- pinned operational directories

The goal is to allow users to quickly move through operational contexts without repeatedly reopening file explorers.

---

## Unified Search and Retrieval

The Connect Library may eventually support unified retrieval across:
- files
- notes
- repos
- snippets
- tasks
- workflows
- views
- apps

The long-term goal is to allow users to search operational context rather than isolated storage locations.

---

## Semantic Retrieval

In the future, the Connect Library may support semantic retrieval.

Examples:
- "show the repo I worked on last night"
- "show the document related to this task"
- "show files related to this workflow"
- "show the screenshots connected to this project"

The goal is to move beyond purely location-based file access.

---

## Resource-Aware Behavior

The Library should remain lightweight whenever possible.

Large indexing operations, previews, and semantic systems should behave intelligently based on:
- system resources
- active workflows
- user activity
- operational priority

The Library should support the broader Connect philosophy of:
- modularity
- lightweight persistence
- workflow continuity
- resource awareness

---

## User Control

Users should remain in control of:
- connected storage systems
- indexed folders
- connected cloud providers
- semantic indexing
- workflow visibility
- app visibility
- retrieval permissions

The Connect Library should feel unified without feeling invasive.

---

## Design Principle

Connect Library is for the user.

Connect is a unified access/orchestration layer, not a data owner.

File Organizer is Alice's janitor.

Bunkers are protected resources visible to Alice only when an approved, scoped, and time-limited session explicitly grants access; absent such a session, Alice may not inspect or operate on Bunker contents for any cognition or execution purpose.

---

## Long-Term Direction

The long-term goal of the Connect Library is to create:
- a unified operational access layer
- a workflow-aware retrieval system
- a contextual workspace navigation system
- an operational memory surface

The larger idea is simple:

The user should not need to remember where something lives.

The workspace should help surface what matters when it matters.

## Related
