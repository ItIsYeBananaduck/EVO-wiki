---
title: EVOconnect — Pane & View System
type: concept
tags: [connect, evo, pane, system, view]
sources:
  - source-materials/mirrors/doctrine/EVOconnect — Pane & View System.md
updated: 2026-07-23
---
# EVOconnect — Pane & View System

## Purpose

The Pane and View System defines how Connect workspaces are assembled, interacted with, restored, suspended, and organized.

The goal of the system is to allow users to create highly customizable workflow environments without forcing them into one rigid application layout.

The Pane and View System should feel:
- modular
- lightweight
- flexible
- persistent
- workflow-oriented
- resource-aware

The system should allow users to move fluidly between lightweight interaction and deep orchestration without losing workspace continuity.

---

# Core Principles

The Pane and View System is built around several core principles.

---

## Panes Are Modular

Every major Connect surface is a pane.

Examples:
- Alice Chat
- EVOterminal
- EVObrowser
- Living Notes
- Scratches
- Tasks
- Git
- EVOcode
- Connect Library

Panes are intended to function independently while also being capable of participating in larger workflow environments.

A pane should never require the existence of a giant parent interface in order to function.

---

## Views Are Workflow Environments

Views are not merely saved layouts.

A view represents:
- a workflow
- a task environment
- an operational state
- a contextual workspace

A view may contain:
- pinned panes
- grouped panes
- temporary panes
- external applications
- workspace rules
- Action Bar configurations

The goal of a view is to restore not only visual arrangement, but workflow continuity.

---

## Workspaces Should Feel Persistent

Connect should avoid feeling disposable or temporary.

The user should feel like:
- workflows sleep instead of disappearing
- workspaces can be resumed
- context is preserved
- state is remembered
- organization persists over time

The system should prioritize continuity over constantly rebuilding workspace state.

---

# Pane Lifecycle

Panes move through several possible states.

---

## Active

An active pane is:
- visible
- interactive
- consuming resources normally
- participating in the current workflow

Active panes may exist:
- independently on the desktop
- inside views
- inside grouped panes
- temporarily above other panes

---

## Collapsed

Collapsed panes reduce visual footprint while retaining lightweight interaction and workflow awareness.

Collapsed panes may:
- retain mini action bars
- expose contextual actions
- expose lightweight workflow state
- remain quickly accessible
- preserve state
- remain attached to their view

Collapsed panes are intended to reduce clutter without destroying workflow continuity.

Collapsed does not necessarily mean suspended.

A collapsed pane may:
- remain lightly active
- continue background monitoring
- continue lightweight Alice workflows
- expose workflow progress
- expose approval requests
- expose lightweight notifications

Examples:
- current Alice task
- current terminal command
- last command output line
- approval required state
- waiting state
- blocked state
- completed state

Collapsed panes are intended to compress the workspace rather than destroy it.

---

## Temporary

Temporary panes are transient workspace surfaces launched from the Action Bar.

Temporary panes:
- pop out temporarily
- perform quick work
- collapse back into the Action Bar when finished
- avoid permanently occupying workspace space

Examples:
- temporary EVOterminal
- temporary notes
- temporary Git access
- temporary browser pane
- temporary library access

Temporary panes allow users to quickly interact with functionality without disrupting their broader workspace layout.

---

## Suspended

Suspended panes release most or all active resource usage while preserving workspace state.

Suspended panes may:
- save state
- release RAM usage
- release processing activity
- disconnect active background behavior
- remain restorable later

The goal is to allow inactive workflows to become dormant instead of continuously consuming system resources.

Suspension behavior may vary depending on:
- pane importance
- active Alice workflows
- user settings
- current system resources
- workflow priority

Examples:
- lightweight task tracking may remain active
- heavy browser workflows may suspend
- grouped development environments may hibernate
- inactive views may fully sleep

---

## Restored

Restored panes return to their previous operational state.

Restoration may include:
- position
- visibility
- pane size
- workflow context
- associated tasks
- related notes
- previous interactions
- temporary state

Restoration should feel seamless and continuous whenever possible.

---

# Pane Behavior

---

## Pinning

Panes may:
- pin to the desktop
- pin into views
- pin into grouped panes
- remain floating independently

Pinned panes become persistent components of their associated workflow.

When a pane becomes pinned:
- its Action Bar button may disappear
- its state becomes associated with the active view
- it becomes part of workspace restoration behavior

---

## Floating

Floating panes operate independently from larger workspace structures.

Floating panes allow:
- quick multitasking
- temporary reference workflows
- lightweight interactions
- cross-view utility access

Floating panes are especially important in Pane Mode.

---

## Docking

Panes may dock into:
- view regions
- grouped pane structures
- workspace layouts

Docking behavior should remain flexible rather than rigidly grid-based whenever possible.

The system should prioritize:
- fluid movement
- user customization
- workflow comfort

over strict interface constraints.

---

# Grouped Panes

Grouped panes allow multiple panes to behave as a unified workspace cluster.

Grouped panes are intended to support:
- IDE-style workflows
- research workflows
- project management stacks
- hybrid operational environments

A grouped pane may internally contain:
- code panes
- terminals
- notes
- Git
- browser panes
- Alice Chat
- tasks

while externally behaving like a larger unified workspace component.

Grouped panes may eventually support:
- unified collapsing
- unified suspension
- unified restoration
- unified movement
- grouped persistence behavior

The purpose of grouped panes is to allow Connect to scale from lightweight workflows to advanced operational environments without becoming visually chaotic.

---

# Views

Views are collections of panes assembled around a workflow.

---

## View Composition

A view may contain:
- pinned panes
- grouped panes
- temporary panes
- floating panes
- external applications
- Action Bar configurations

Views are intended to allow users to rapidly switch operational environments.

---

## View Restoration

When a view restores, Connect may restore:
- pane layouts
- grouped panes
- pinned states
- collapsed states
- suspended states
- external applications
- workspace context
- workflow continuity

Restoration should prioritize minimizing friction and reducing manual workspace reconstruction.

---

## Default Views

Users may designate default views.

Default views restore automatically when entering Command Center Mode or specific Spaces.

Different Spaces may have different default views.

---

# Action Bar

The Action Bar acts as the orchestration layer for panes and views.

The Action Bar is responsible for:
- launching panes
- restoring panes
- temporarily opening panes
- restoring views
- exposing lightweight interactions
- exposing workflow state
- exposing lightweight notifications
- reducing workspace clutter
- maintaining workflow continuity

The Action Bar should remain persistent even when larger workspaces collapse or suspend.

The Action Bar is not only a launcher. It is also a lightweight operational surface for monitoring ongoing workflow activity.

Examples:
- current Alice task
- current terminal command
- active workflow status
- approval requests
- waiting states
- task completion notifications
- blocked workflow notifications

This allows users to remain aware of ongoing Alice activity without requiring the full workspace to remain visible.

---

## Temporary Pane Orchestration

One of the Action Bar’s primary responsibilities is temporary pane orchestration.

Users should be able to:
- quickly open a pane
- perform a small amount of work
- collapse the pane back into the Action Bar
- continue their primary workflow uninterrupted

This allows users to access important functionality without permanently dedicating workspace space to every pane.

---

## Action Bar Clutter Reduction

The Action Bar should dynamically reduce clutter.

Examples:
- pinned panes may disappear from the Action Bar
- inactive workflows may collapse
- grouped panes may consolidate controls
- temporary panes may disappear after use

The Action Bar should feel adaptive and workflow-aware rather than static.

---

# Mini Action Bars

Mini action bars are lightweight contextual controls attached to collapsed panes.

Mini action bars allow users to perform quick contextual interactions without reopening the full pane.

Mini action bars may also expose lightweight live workflow state.

Examples:
- quick add task
- quick note capture
- paste to note
- quick scratch
- rerun command
- save browser content
- current workflow status
- approval needed indicators
- active Alice task status

Mini action bars are intended to reduce workflow interruption and preserve focus while still keeping users aware of important operational activity.

---

# Spaces and Views

Spaces act as higher-level workflow domains above views.

Changing Spaces may change:
- available views
- Action Bar configurations
- pinned applications
- quick actions
- pane availability
- workspace priorities
- workflow behaviors

This allows users to maintain completely different operational environments for different categories of work.

Examples:
- Life Space
- Work Space
- Development Space
- Research Space

The goal is for users to feel like they are entering different operational environments rather than merely switching tabs or windows.

---

# EVOcode and Pane Composition

EVOcode is not intended to function as a monolithic IDE.

EVOcode is a code-aware pane that participates in broader pane composition workflows.

The Connect workspace itself becomes the IDE.

Users may assemble:
- EVOcode
- EVOterminal
- Git
- Notes
- Tasks
- Browser panes
- Alice Chat

into custom development environments shaped around their preferred workflow.

This allows users to create:
- lightweight development views
- advanced IDE-style views
- hybrid research/development environments
- project management coding workflows

using the same underlying workspace system.

---

## IDE Composition Philosophy

Traditional IDEs are typically large monolithic applications with tightly coupled tooling.

Connect instead approaches development workflows through pane composition.

Development environments are assembled from independent panes that may:
- operate independently
- suspend independently
- restore independently
- reorganize dynamically
- participate in multiple workflows

This allows users to create:
- lightweight coding environments
- advanced development workspaces
- temporary debugging workflows
- hybrid research and development environments

without requiring one giant permanently active IDE process.

Grouped panes may eventually allow users to create reusable development clusters composed of multiple interconnected panes.

---

# Resource-Aware Workspace Behavior

Connect should continuously remain aware of:
- current system resources
- active workflows
- active Alice tasks
- pane importance
- workflow priority

Alice may:
- recommend suspending workflows
- recommend closing applications
- recommend hibernating views
- prioritize lightweight operational surfaces
- preserve important workflows during resource pressure

Examples:
- suspending inactive browser panes
- hibernating unused development views
- preserving lightweight task monitoring
- keeping Alice active while heavier workflows sleep

The goal is to maximize workflow continuity while minimizing unnecessary resource usage.

---

# Long-Term Direction

The long-term goal of the Pane and View System is to allow Connect to scale naturally between:
- lightweight interactions
- deep operational work
- complex multitasking
- resource-sensitive workflows
- AI-assisted orchestration

without losing:
- modularity
- persistence
- workflow continuity
- contextual awareness
- resource efficiency

The Pane and View System should ultimately make Connect feel less like a traditional application and more like a living workspace environment that adapts around the user’s workflow.

## Related
- [[EVOconnect — System Map]]
- [[EVO Architecture Bible]]
- [[EVOconnect — Action Bar & Mini Action Bar System.md]]
- [[EVOconnect — Coach Pane Pack Contract.md]]
- [[EVOconnect — Connect Library & Unified Access Layer.md]]
- [[EVOconnect — Hive Node Architecture.md]]
- [[EVOconnect — Lightweight Talent Structure Addendum.md]]
- [[EVOconnect — Method Reconstruction Model.md]]
^[source-materials/mirrors/doctrine/EVOconnect — Pane & View System.md]
