---
title: EVOconnect — Workspace Architecture (Raw Draft v2)
type: concept
tags: [architecture, connect, evo, workspace]
sources:
  - source-materials/mirrors/doctrine/EVOconnect — Workspace Architecture.md
updated: 2026-07-23
---
# EVOconnect — Workspace Architecture (Raw Draft v2)

## Purpose

EVOconnect is not intended to be a single giant productivity app. It is intended to function as a modular AI workspace layer that sits on top of the user’s existing operating system and workflows.

The goal is not to replace macOS, Windows, or Linux. The goal is to orchestrate workflows, cognition, workspace state, and resource management in a way that traditional operating systems and productivity applications currently do not.

Connect should feel less like one massive application and more like a coordinated bundle of intelligent workspace components that the user can shape around their own workflow.

---

## Core Principle

Connect is a workspace operating layer, not a monolithic application.

Traditional operating systems manage:
- processes
- windows
- files
- hardware resources

Connect manages:
- workflows
- spaces
- modes
- views
- panes
- workspace state
- cognitive context
- workflow continuity
- resource-aware behavior
- AI orchestration

Alice acts as the persistent cognitive layer across the workspace while the workspace itself dynamically changes around the user’s current workflow.

---

## Connect Philosophy

The user should not feel trapped inside a giant application window.

Instead, Connect should feel like a persistent workspace environment that can:
- expand
- collapse
- reorganize
- suspend
- restore
- adapt

depending on what the user is currently trying to accomplish.

The workspace itself should feel alive and modular.

---

# Workspace Hierarchy

Connect organizes the workspace into several layers.

```text
Connect
→ Spaces
→ Modes
→ Views
→ Panes
```

Each layer exists for a specific purpose.

---

## Spaces

Spaces represent the user’s highest-level workflow domains.

Examples:
- Life Space
- Work Space
- Development Space
- Research Space

Changing spaces changes the broader operational environment.

A space may contain:
- different views
- different Action Bars
- different workflows
- different pane configurations
- different pinned applications
- different quick actions
- different workspace priorities

The goal is for users to feel like they are switching operational contexts rather than simply changing windows.

---

## Modes

Connect currently consists of three primary interaction modes.

---

### Pane Mode

Pane Mode is the lightweight workspace mode.

In this mode:
- panes may exist independently on the desktop
- panes may float, pin, collapse, or temporarily hide
- collapsed panes retain lightweight functionality through mini action bars
- mini action bars expose quick contextual actions without requiring the full pane to open

Examples:
- quick add task
- quick scratch note
- paste to note
- rerun terminal command
- quick browser save

Pane Mode is intended to keep Connect present without overwhelming the user’s desktop.

---

### Command Center Mode

Command Center Mode is the advanced orchestration workspace.

When entering Command Center Mode:
- the HUD collapses into the taskbar
- the Action Bar slides out
- panes may be launched, pinned, grouped, restored, or temporarily opened

Users create workflow environments called views.

A view is composed of panes arranged around a specific workflow.

Examples:
- Development View
- Research View
- Planning View
- Streaming View
- Writing View

Views may restore:
- pane positions
- pinned panes
- grouped panes
- external applications
- workspace state
- workflow context

Views may also be designated as default views.

The goal of Command Center Mode is to allow users to build highly customized workflow environments without requiring one giant monolithic interface.

---

### Connect HUD Mode

HUD Mode is the lightweight persistent access layer.

The HUD remains available even when the larger workspace is collapsed or suspended.

The HUD provides:
- quick chat access
- notifications
- favorites
- shortcuts
- lightweight actions
- quick workspace access

The HUD is intended to keep Connect accessible without requiring the full workspace to remain active.

---

## Panes

Panes are the primary modular workspace units inside Connect.

Panes are intended to be fully modular and independently manageable.

A pane may:
- exist independently
- pin to the desktop
- pin inside a view
- collapse into the Action Bar
- temporarily appear
- suspend
- restore
- remain persistent
- participate in saved views
- participate in grouped pane compositions

Connect core ships with base panes while additional panes may be introduced later as add-ons.

---

## Views

Views are saved workspace configurations composed of panes.

A view represents a workflow environment rather than merely a visual layout.

A view may contain:
- individual panes
- grouped panes
- temporary panes
- external applications
- Action Bar configurations

Views are intended to reduce workspace fragmentation and allow rapid workflow switching.

---

## Grouped Panes

Panes may eventually be grouped together into larger pane compositions.

A grouped pane behaves like a single operational workspace while internally containing multiple panes.

Example:
A development group may internally contain:
- EVOcode
- EVOterminal
- Git
- Alice Chat
- Notes

while behaving externally as a single workspace cluster.

This allows Connect to function as:
- an IDE
- a research environment
- a project management environment
- a planning workspace
- a lightweight productivity system

depending entirely on how the user assembles their panes and views.

---

## Action Bar

The Action Bar acts as the orchestration layer for panes and views.

The Action Bar:
- launches panes
- restores panes
- restores views
- temporarily opens panes
- manages workspace visibility
- provides lightweight pane interaction

The Action Bar is not only a launcher. It is also a temporary workspace interaction layer.

Temporary panes may:
- pop out
- perform quick work
- collapse back into the Action Bar
- avoid permanently occupying workspace space

Examples:
- temporary EVOterminal access
- temporary browser access
- temporary notes access
- temporary Git access
- temporary library access

When panes are pinned into a view, their buttons may disappear from the Action Bar to reduce clutter.

The Action Bar remains persistent even when larger views are collapsed or suspended.

---

## Mini Action Bars

Collapsed panes may retain mini action bars.

Mini action bars expose lightweight contextual actions without requiring the pane to reopen fully.

Examples:
- quick add task
- quick create note
- quick scratch capture
- paste to note
- rerun recent command
- quick browser clip

Mini action bars are intended to reduce friction and allow users to perform quick contextual interactions while remaining focused on their current work.

---

## Workspace Resource Management

One of Connect’s primary goals is resource-aware workflow orchestration.

Traditional workflows often leave large numbers of applications running simultaneously, consuming RAM and CPU resources even when inactive.

Connect introduces workflow hibernation.

When switching workflows or entering resource-sensitive situations:
- panes may suspend
- grouped panes may suspend
- views may collapse into the Action Bar
- external applications may save and close
- dormant workflows may release resources
- Alice may remain active while the rest of the workspace sleeps

The goal is to allow workflows to be restored later without requiring users to manually reconstruct their environment.

---

## Alice’s Role

Alice acts as the persistent cognitive layer across all Connect modes.

Alice should remain aware of:
- active spaces
- active modes
- active views
- active panes
- workflow context
- related tasks
- related notes
- workspace state

Alice should adapt her context depending on the pane or workflow the user is interacting with.

Examples:
- opening development panes loads development context
- opening notes loads note-related context
- opening research panes loads browser and note context

Alice should feel continuously present across the workspace rather than trapped inside a single application window.

Alice may eventually:
- auto-avoid workspace collisions
- reposition intelligently
- follow active workflows contextually

while remaining a consistent visual and cognitive presence across the workspace.

---

## Third-Party Applications

Connect may eventually support integrating third-party applications into views.

External applications may:
- launch with a view
- restore with a view
- suspend with a view
- close with a view
- participate in workflow orchestration

This allows Connect to act as a coordination layer around the user’s broader workflow ecosystem rather than forcing users into a completely isolated environment.

---

## EVOcode

EVOcode is not intended to be a monolithic IDE replacement.

EVOcode is a code-aware pane.

Architecturally, EVOcode is closer to Living Notes and Scratches than to a traditional IDE.

The primary difference is:
- syntax awareness
- code-oriented workflows
- development-oriented formatting

EVOcode acts primarily as:
- a code editing pane
- a visualization layer
- a collaborative development surface
- a manual intervention layer

The Connect workspace itself becomes the IDE through pane composition.

A development workflow may combine:
- EVOcode
- EVOterminal
- Git
- Alice Chat
- Notes
- Tasks
- Browser panes

to create a custom development environment shaped entirely around the user’s preferred workflow.

This allows users to create:
- lightweight IDEs
- advanced IDEs
- research environments
- planning systems
- hybrid workflows

using the same underlying workspace architecture.

Future versions may support:
- optional extensions
- smart comments
- contextual code conversations
- educational workflow assistance

---

## Long-Term Direction

The long-term goal of Connect is to create a workspace environment where:
- workflows are persistent
- cognition is contextual
- resource usage is intentional
- panes are modular
- views are workflow-oriented
- workflows are resumable
- workspace state is preserved
- Alice remains continuously aware of the user’s operational state

Connect should ultimately feel like:
- a workspace operating layer
- an AI orchestration shell
- a cognitive workflow environment

rather than simply another productivity application.

## Related
- [[EVOconnect — System Map]]
- [[EVO Architecture Bible]]
- [[EVOconnect — Action Bar & Mini Action Bar System.md]]
- [[EVOconnect — Coach Pane Pack Contract.md]]
- [[EVOconnect — Connect Library & Unified Access Layer.md]]
- [[EVOconnect — Hive Node Architecture.md]]
- [[EVOconnect — Lightweight Talent Structure Addendum.md]]
- [[EVOconnect — Method Reconstruction Model.md]]
^[source-materials/mirrors/doctrine/EVOconnect — Workspace Architecture.md]
