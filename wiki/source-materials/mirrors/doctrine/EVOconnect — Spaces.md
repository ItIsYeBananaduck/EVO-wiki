---
title: "EVOconnect — Spaces (Raw Draft)"
type: doctrine
tags: ['lsctech', 'doctrine', 'source-material', 'evo']
updated: 2026-08-19
---

# EVOconnect — Spaces (Raw Draft)

## Purpose

Spaces define the highest-level operational domains inside Connect.

The goal of Spaces is to prevent Connect from becoming one giant universal workspace where every workflow, pane, shortcut, and operational state competes for attention.

Spaces allow users to separate workflows into meaningful operational environments.

Examples:
- Life Space
- Work Space
- Development Space
- Research Space

A Space should feel like entering a different operational context rather than merely switching windows or tabs.

---

# Core Principle

Spaces define operational context.

Views define workflow environments.

Panes define workspace units.

This means:
- Spaces answer “what kind of work am I doing?”
- Views answer “what workflow am I using?”
- Panes answer “what tools are active?”

This hierarchy allows Connect to scale without becoming visually or cognitively chaotic.

---

# Relationship to the Workspace Hierarchy

```text
Connect
→ Spaces
→ Modes
→ Views
→ Panes
```

Spaces sit at the top of the operational workspace hierarchy.

Changing Spaces may affect:
- available views
- Action Bar configuration
- pane availability
- pinned applications
- workflow visibility
- Alice context
- resource behavior
- notifications
- operational priorities

---

# Space Identity

Each Space represents a different operational environment.

A Space may contain:
- its own views
- its own Action Bar state
- its own workflows
- its own pinned panes
- its own grouped panes
- its own temporary panes
- its own external applications

The goal is for each Space to feel purpose-built for its operational domain.

---

# Space Examples

## Life Space

Life Space may prioritize:
- tasks
- reminders
- journaling
- quick capture
- lightweight workflows
- personal organization

The interface should feel lightweight and calm.

---

## Work Space

Work Space may prioritize:
- operational tasks
- project management
- workflow tracking
- approvals
- communication workflows
- persistent operational visibility

The interface may feel more workflow-dense.

---

## Development Space

Development Space may prioritize:
- EVOcode
- EVOterminal
- Git
- development views
- debugging workflows
- implementation tasks
- repo context

This Space may expose more development-oriented operational behavior.

---

## Research Space

Research Space may prioritize:
- EVObrowser
- notes
- references
- clipping workflows
- source tracking
- lightweight summarization

This Space may optimize for information gathering and contextual organization.

---

# Space-Specific Action Bars

Each Space may have its own Action Bar configuration.

Different Spaces may expose:
- different shortcuts
- different temporary panes
- different quick actions
- different workflow visibility
- different notifications

Examples:

Development Space:
- terminal shortcuts
- Git workflows
- snippets
- repo access

Life Space:
- reminders
- quick notes
- journaling
- lightweight task capture

This allows the Action Bar to remain contextual instead of becoming overloaded.

---

# Space-Specific Views

Views belong to Spaces.

Different Spaces may contain:
- different default views
- different grouped panes
- different workflow layouts
- different operational priorities

Examples:
- Development Space may default into a coding view
- Research Space may default into a browser and notes workflow
- Life Space may default into a lightweight dashboard

This helps each Space feel cohesive and intentional.

---

# Space Transitions

Switching Spaces should feel smooth and operationally aware.

When changing Spaces, Connect may:
- collapse inactive views
- suspend unrelated panes
- hibernate inactive workflows
- restore Space-specific views
- restore Space-specific Action Bars
- change Alice context
- restore relevant operational state

The goal is for users to feel like they are moving between operational environments, not rebooting applications.

---

# Alice Across Spaces

Alice should understand the active Space.

Different Spaces may change:
- Alice context priorities
- active workflows
- visible tasks
- available panes
- suggested actions
- operational focus

Examples:
- Development Space may bias Alice toward implementation workflows
- Research Space may bias Alice toward summarization and note linkage
- Life Space may bias Alice toward reminders and lightweight organization

Alice should feel contextually aware without requiring the user to manually restate their operational intent every time they switch workflows.

---

# Resource Behavior Per Space

Different Spaces may have different resource expectations.

Examples:

Life Space:
- lightweight persistence
- minimal background activity
- lower operational intensity

Development Space:
- heavier workflows
- grouped pane orchestration
- terminal workflows
- repo indexing
- more active Alice orchestration

Spaces may influence:
- suspension behavior
- hibernation behavior
- background workflows
- extension activation
- resource prioritization

---

# Notifications and Operational Visibility

Spaces may control which notifications and operational states are prioritized.

Examples:
- Development Space may prioritize terminal status and task progress
- Life Space may prioritize reminders and journaling prompts
- Research Space may prioritize saved references and clipped content

This prevents operational noise from unrelated workflows.

---

# Long-Term Direction

The long-term goal of Spaces is to allow Connect to scale into a large ecosystem without becoming cognitively overwhelming.

Spaces should make Connect feel like:
- multiple operational environments
- one continuous workspace system
- contextual workflow domains
- adaptive cognitive environments

rather than:
- one giant productivity dashboard
- one overloaded universal workspace

The larger idea is simple:

The user should feel like they are entering the right environment for the work they are trying to do.
