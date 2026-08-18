# EVOconnect — Modes (Raw Draft)

## Purpose

Modes define how the user interacts with the Connect workspace.

The goal of the mode system is to allow Connect to scale naturally between:
- lightweight interaction
- focused workflows
- deep orchestration
- background AI assistance
- complex multitasking

without forcing the user into one rigid interface style.

Modes are not separate applications.

They are different operational states of the same workspace system.

---

# Core Principle

Modes define interaction style, not functionality.

The same panes, views, workflows, and Alice systems may exist across all modes.

What changes is:
- visibility
- orchestration behavior
- interaction density
- workspace complexity
- resource behavior
- operational focus

The goal is to allow the workspace to adapt around the user instead of forcing the user to adapt around the workspace.

---

# Mode Hierarchy

Connect currently contains three primary modes.

```text
Connect
→ Spaces
→ Modes
→ Views
→ Panes
```

Modes sit below Spaces and above Views.

This means:
- Spaces define operational domains
- Modes define interaction style
- Views define workflow environments
- Panes define workspace units

---

# Pane Mode

## Purpose

Pane Mode is the lightweight workspace mode.

It is intended for:
- casual multitasking
- lightweight workflow interaction
- passive monitoring
- quick access workflows
- keeping Connect available without overwhelming the desktop

Pane Mode should feel:
- lightweight
- minimal
- persistent
- flexible
- unobtrusive

---

## Pane Behavior

In Pane Mode:
- panes may float independently
- panes may pin to the desktop
- panes may collapse into mini action bars
- panes may temporarily appear and disappear
- panes may remain lightly active in the background

Pane Mode prioritizes flexibility and lightweight interaction over dense orchestration.

---

## Mini Action Bars

Mini action bars are especially important in Pane Mode.

Collapsed panes may continue exposing:
- quick actions
- workflow state
- lightweight notifications
- approval requests
- current Alice activity

Examples:
- current task
- current terminal command
- quick add task
- quick scratch note
- approval required
- workflow complete

Mini action bars allow users to stay aware of ongoing workflows without requiring full pane visibility.

---

## Alice in Pane Mode

In Pane Mode, Alice acts as a lightweight persistent assistant presence.

Alice may:
- remain visible
- remain movable
- avoid UI collisions
- monitor lightweight workflows
- surface approval requests
- provide contextual awareness

Pane Mode is intended to keep Alice available without demanding constant attention.

---

# Command Center Mode

## Purpose

Command Center Mode is the advanced orchestration workspace.

This is the primary environment for:
- deep work
- complex workflows
- multitasking
- development workflows
- research workflows
- project management
- AI-assisted operational work

Command Center Mode should feel:
- powerful
- modular
- customizable
- workflow-oriented
- operationally dense

without becoming visually chaotic.

---

## Entering Command Center

When entering Command Center:
- the HUD collapses into the taskbar
- the Action Bar slides out
- views become available
- panes may restore
- grouped panes may restore
- workspace state may restore

The transition should feel like entering a deeper operational layer of the workspace.

---

## Views in Command Center

Command Center revolves around views.

Views define:
- pane arrangement
- grouped panes
- temporary panes
- Action Bar configuration
- external applications
- workflow state

Users may:
- create views
- save views
- restore views
- duplicate views
- assign default views
- organize views per Space

Command Center should allow users to build personalized operational environments around their workflow.

---

## Grouped Panes

Grouped panes are especially important in Command Center.

Grouped panes allow multiple panes to behave as:
- a unified operational cluster
- a development environment
- a research stack
- a project management environment

Examples:
- IDE-style pane groups
- research groups
- planning groups
- hybrid workflow groups

Grouped panes allow Command Center to scale into advanced workflows without requiring monolithic applications.

---

## Action Bar in Command Center

The Action Bar acts as the orchestration layer for Command Center.

The Action Bar:
- launches panes
- restores panes
- restores views
- temporarily opens panes
- exposes workflow state
- surfaces approvals
- reduces workspace clutter

Temporary panes may:
- pop out temporarily
- perform quick work
- collapse back into the Action Bar
- avoid permanently occupying workspace space

The Action Bar remains persistent even when views collapse or suspend.

---

## Alice in Command Center

In Command Center, Alice acts as the operational coordinator.

Alice should understand:
- active views
- active panes
- active workflows
- resource state
- task progress
- approval requirements
- workflow relationships

Alice may:
- assist workflows
- monitor workflows
- coordinate workflows
- negotiate resource usage
- recommend suspension behavior
- surface operational status

Command Center is where Alice’s orchestration role becomes most visible.

---

# Connect HUD Mode

## Purpose

Connect HUD Mode is the lightweight persistent access layer.

HUD Mode exists so Connect remains:
- accessible
- aware
- responsive
- operationally present

without requiring the larger workspace to remain visible.

HUD Mode should feel:
- fast
- lightweight
- minimal
- persistent
- always available

---

## HUD Behavior

The HUD may contain:
- quick chat access
- notifications
- favorites
- shortcuts
- quick actions
- lightweight workflow visibility

The HUD is intended to function as:
- a lightweight launcher
- a notification surface
- a quick interaction layer
- a persistent operational presence

---

## Relationship to Command Center

The HUD and Command Center are closely connected.

Entering Command Center:
- collapses the HUD
- expands the Action Bar
- restores deeper workspace orchestration

Leaving Command Center:
- may collapse views
- may suspend panes
- may hibernate workflows
- restores lightweight HUD interaction

This transition should feel smooth and continuous rather than like switching between separate applications.

---

## Alice in HUD Mode

In HUD Mode, Alice acts as:
- a lightweight assistant
- a persistent awareness layer
- a notification surface
- a quick interaction point

Alice may:
- provide lightweight updates
- surface approvals
- summarize workflows
- expose ongoing status
- remain available for quick interaction

HUD Mode prioritizes accessibility and continuity over deep workspace interaction.

---

# Cross-Mode Continuity

The workspace should feel continuous across all modes.

The user should not feel like:
- they are reopening apps
- they are rebuilding layouts
- they are changing systems

Instead, modes should feel like:
- different interaction depths
- different operational intensities
- different workspace states

of the same persistent workspace environment.

---

# Resource Behavior Across Modes

Different modes may prioritize resources differently.

Pane Mode:
- prioritizes lightweight persistence

Command Center:
- prioritizes operational capability

HUD Mode:
- prioritizes minimal footprint

Connect should adapt resource behavior depending on:
- current mode
- current workflows
- system resources
- active Alice tasks
- user preferences

---

# Long-Term Direction

The long-term goal of the mode system is to allow Connect to scale naturally between:
- passive awareness
- lightweight interaction
- deep operational work
- AI-assisted orchestration
- complex multitasking

while maintaining:
- continuity
- modularity
- persistence
- contextual awareness
- resource efficiency

Modes should ultimately make Connect feel less like switching applications and more like changing the operational depth of the same living workspace environment.
