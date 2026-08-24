---
title: EVOconnect — Connect Library & Unified Access Layer (Raw Draft)
type: concept
tags: [connect, evo]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# EVOconnect — Connect Library & Unified Access Layer (Raw Draft)

## Purpose

The Connect Library defines how users and Alice access the broader workspace ecosystem.

The Connect Library is not intended to function as:
- a simple file browser
- a traditional launcher
- a disconnected storage explorer

The Connect Library is intended to function as:
- a unified access layer
- a workspace retrieval system
- an operational indexing layer
- a workflow access system
- a unified storage abstraction

The goal is to give both the user and Alice one coherent way to navigate:
- files
- folders
- repos
- apps
- workflows
- panes
- views
- Spaces
- notes
- tasks
- connected storage systems

without fragmenting the workspace into disconnected systems.

---

# Core Principle

The user should not need to think about where something is stored.

The user should think about:
- what they are working on
- what project they need
- what workflow they want
- what context matters right now

The Connect Library should unify access across:
- local storage
- cloud storage
- external drives
- network storage
- workspace systems
- operational workflows

into one coherent workspace access layer.

---

# Unified Storage View

The Connect Library should expose a unified view of all connected storage systems.

Examples:
- local files
- iCloud
- Google Drive
- Dropbox
- OneDrive
- NAS storage
- external drives
- synced workspace storage

The goal is not to force users into one storage provider.

The goal is to allow users to work across multiple storage systems without constantly switching applications or file explorers.

---

# Unified Operational Access

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

This allows the Library to become:
- an operational retrieval layer
- a workflow discovery system
- a workspace coordination surface

rather than only a filesystem viewer.

---

# Relationship to Spaces

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

The Connect Library should adapt contextually to the active operational environment.

---

# Relationship to Views

Views may contain Library-driven workflows.

Examples:
- pinned project folders
- docked repos
- quick-access research folders
- workflow-linked directories
- temporary file access panes

The Library should feel tightly integrated into workspace organization.

---

# Dockable Folders

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

# Dockable Repositories

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

# Temporary File Panes

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

# Workflow-Aware Retrieval

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

# Relationship to Alice

Alice should understand:
- active folders
- active repos
- active workflows
- active files
- workspace relationships
- operational context

Alice may:
- recommend relevant folders
- restore workflows
- reopen operational contexts
- suggest related files
- surface related notes
- connect related tasks

This allows Alice to participate in the user’s broader operational environment rather than existing separately from it.

---

# Action Bar Integration

The Action Bar should integrate tightly with the Connect Library.

Examples:
- docked folders
- docked repos
- quick-access workflows
- temporary file panes
- pinned operational directories

The goal is to allow users to quickly move through operational contexts without repeatedly reopening file explorers.

---

# Unified Search and Retrieval

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

# Semantic Retrieval

In the future, the Connect Library may support semantic retrieval.

Examples:
- “show the repo I worked on last night”
- “show the document related to this task”
- “show files related to this workflow”
- “show the screenshots connected to this project”

The goal is to move beyond purely location-based file access.

---

# Resource-Aware Behavior

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

# User Control

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

# Long-Term Direction

The long-term goal of the Connect Library is to create:
- a unified operational access layer
- a workflow-aware retrieval system
- a contextual workspace navigation system
- an operational memory surface

The larger idea is simple:

The user should not need to remember where something lives.

The workspace should help surface what matters when it matters.

## Related
- [[EVOconnect — System Map]]
- [[EVO Architecture Bible]]
- [[EVOconnect — Action Bar & Mini Action Bar System.md]]
- [[EVOconnect — Coach Pane Pack Contract.md]]
- [[EVOconnect — Hive Node Architecture.md]]
- [[EVOconnect — Lightweight Talent Structure Addendum.md]]
- [[EVOconnect — Method Reconstruction Model.md]]
- [[EVOconnect — Mobile Operational Continuity.md]]
^[wiki-native — no upstream source]
