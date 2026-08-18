---
title: EVO — Pane Pack Architecture (Raw Draft)
type: concept
tags: [architecture, evo, pane]
sources:
  - source-materials/mirrors/doctrine/EVO — Pane Pack Architecture.md
updated: 2026-07-23
---
# EVO — Pane Pack Architecture (Raw Draft)

## Purpose

This note defines the Pane Pack architecture for the EVO ecosystem.

Pane Packs exist so EVO apps can share reusable operational surfaces without forcing every app to become Connect or every domain system to be rebuilt multiple times.

The goal is to allow EVO domain apps to expose useful panes that can later be hosted, composed, grouped, and orchestrated inside Connect.

This creates a shared architecture where:
- standalone apps remain focused
- Connect becomes the advanced orchestration host
- domain-specific functionality can travel across EVO
- users can build workflows from reusable operational pieces

---

# Core Principle

EVO apps should not be treated as monolithic blocks inside Connect.

Instead, domain apps may expose Pane Packs.

A Pane Pack is a portable operational package that contains:
- reusable panes
- domain workflows
- domain talents
- domain logic
- orchestration bindings
- pane contracts
- Alice capability hooks
- state management rules

The important distinction is:

- EVOtraining owns training logic.
- EVOtraining Coach exposes coaching panes through a Coach Pane Pack.
- Connect hosts those panes when the user wants advanced composition.

Connect does not need the entire app.

Connect needs approved, portable operational surfaces.

---

# Pane Packs vs Apps

An EVO app is a full domain experience.

A Pane Pack is a reusable set of portable operational systems from that domain.

For example:

EVOtraining may include:
- mobile workout execution
- wearable integration
- user workout flow
- nutrition tracking
- trainer relationships
- coach systems

But the Coach Pane Pack may expose:
- client list
- client profile
- plan builder
- client analytics
- progress reports
- coach messaging
- billing or marketplace surfaces
- review queues
- coaching talents
- coaching workflows
- plan orchestration logic

The Pane Pack does not need to expose every screen from the app.

Only systems that are useful outside the home app should become portable.

---

# Pane Packs are Self-Contained

Pane Packs should not require the full standalone app shell to function.

This is extremely important architecturally.

A Pane Pack should carry:
- its pane UI
- its domain workflows
- its talents
- its orchestration logic
- its state logic
- its domain actions
- its Connect integration hooks

This allows Connect to host the Pane Pack directly without requiring the standalone app to remain installed.

The standalone app becomes:
- a focused shell
- a guided host
- a simplified operational experience

not the source of truth for the functionality itself.

---

# Shared Runtime Layer

Pane Packs may still depend on shared EVO runtime systems.

Examples:
- Alice runtime
- Hive
- authentication
- sync
- storage
- Connect Library access
- shared AI runtimes
- shared orchestration APIs
- shared design systems

These are ecosystem-level runtime systems, not app-specific dependencies.

This distinction keeps Pane Packs portable and modular.

---

# Connect as Pane Host

Connect acts as the advanced pane host.

Inside Connect, Pane Packs may:
- be installed
- be activated
- expose pane buttons
- appear in the Action Bar
- participate in Views
- participate in Spaces
- group with other panes
- suspend or hibernate with workflows
- connect to Alice workflows

This allows Connect users to combine domain-specific panes with Connect core panes.

Example:

A coach inside Connect may combine:
- Client Analytics pane
- Plan Builder pane
- Alice Chat pane
- Living Notes pane
- Tasks pane
- Calendar pane
- EVObrowser pane

to create a custom coaching workflow.

---

# Standalone Apps as Locked Hosts

Standalone EVO apps may also host Pane Packs.

The difference is that standalone apps use locked or guided pane layouts.

For example:

EVOtraining Coach standalone app may use the Coach Pane Pack as a fixed grouped-pane workspace.

In the standalone app:
- panes may not detach
- pane layout may be guided
- workflows may be simplified
- only domain-relevant panes appear
- advanced Connect composition controls are hidden

This keeps standalone apps approachable for users who do not need full Connect power.

---

# Connect Unlocks Pane Composition

When the same Pane Pack is used inside Connect, the panes become more flexible.

Inside Connect, users may:
- detach panes
- pin panes
- group panes
- resize panes
- combine panes with other domains
- save panes into Views
- add panes to Spaces
- use panes with the Action Bar
- hibernate pane groups

This creates an upgrade path.

A user can start with a focused standalone domain app and later move into Connect without losing familiar tools.

The workflow evolves instead of being rebuilt.

---

# Shared Base Panes

Pane Packs should not duplicate Connect core panes unless they have a strong domain-specific reason.

Connect already provides base panes such as:
- Alice Chat
- Tasks
- Living Notes
- Scratches
- Connect Library
- EVObrowser
- EVOterminal

A Pane Pack should only provide domain-specific panes.

For example, the Coach Pane Pack does not need its own generic notes pane if Living Notes already exists.

It may instead provide:
- client note context
- coaching review context
- plan-specific annotations

using the existing Notes infrastructure.

---

# Pane Ownership

Domain apps own their domain logic.

Pane Packs expose selected surfaces from that logic.

Connect hosts the pane, but it does not own the domain.

For example:
- EVOtraining owns training data, training logic, coaching rules, and plan adaptation.
- Connect hosts coaching panes and helps orchestrate workflows around them.
- Alice may coordinate across both, but permissions and data ownership remain domain-scoped.

This prevents Connect from becoming a tangled copy of every EVO app.

---

# Pane Contract

Each portable pane should define a clear contract.

A pane contract may include:
- pane identity
- domain owner
- required permissions
- data dependencies
- supported host apps
- supported modes
- allowed actions
- save and restore behavior
- suspension behavior
- required Alice capabilities
- whether the pane can detach
- whether the pane can be grouped
- whether the pane can appear in the Action Bar

The contract keeps panes portable without making them ambiguous.

---

# Host Permissions

Not every host can use every pane.

A pane may support:
- standalone app host
- Connect desktop host
- Connect tablet dashboard host
- Connect mobile host
- enterprise host
- read-only host
- approval-only host

For example:
- Plan Builder may work in Coach standalone and Connect desktop.
- Client Analytics may work in Coach standalone, Connect desktop, and tablet dashboard.
- Mobile may only expose lightweight review or approval surfaces.

This prevents desktop-only workflows from being forced onto mobile.

---

# Shared vs App-Specific Panes

Panes should be shared only when they are useful outside their home app.

Good candidates for shared panes:
- analytics panes
- planning panes
- client/profile panes
- review panes
- dashboard panes
- workflow status panes
- approval panes

Poor candidates for shared panes:
- highly device-specific flows
- mobile-only workout execution UI
- watch-specific screens
- onboarding screens
- settings screens that only make sense in the source app
- temporary local UI flows

The goal is portability without bloating Connect.

---

# Pane Groups

Pane Packs may include recommended pane groups.

A pane group is a named composition of panes that behaves like a cohesive operational unit.

Examples:
- Coach Dashboard
- Client Review Workspace
- Plan Builder Workspace
- Live Session Analytics Workspace

In standalone apps, these groups may be locked.

In Connect, these groups may be detachable and customizable.

This allows domain apps to provide polished default workflows while still allowing Connect users to customize them.

---

# Resource Behavior

Pane Packs must participate in Connect resource orchestration.

Portable panes should support:
- collapse
- suspend
- restore
- save state
- release heavy resources when inactive
- expose lightweight status when appropriate

This matters because Connect’s power comes from allowing users to compose many workflows without overwhelming the machine.

Pane Packs must respect that design.

---

# Alice and Pane Packs

Alice should understand Pane Packs as domain-specific capability surfaces.

Alice may:
- open panes
- explain panes
- use pane actions
- summarize pane state
- coordinate pane workflows
- suggest relevant panes
- help build Views from panes

However, Alice must respect:
- pane permissions
- domain boundaries
- enterprise scope
- user approval requirements

Alice should not treat access to a pane as unlimited access to the entire domain app.

---

# Enterprise and Scoped Pane Access

Pane Packs are especially important for enterprise workflows.

An enterprise may expose approved Pane Packs to users.

Those panes may be:
- scoped to company data
- restricted by role
- unavailable on personal devices
- revoked when access ends
- visible only inside enterprise Spaces

This allows the same Connect architecture to support both personal and enterprise workflows without mixing data ownership.

---

# Host and Runtime Model

The EVO ecosystem should follow a layered host model.

```text
EVO Runtime Layer
  ↓
Shared Core Systems
  ↓
Pane Packs
  ↓
Hosts
```

Possible hosts include:
- Connect
- Coach standalone
- Learn standalone
- dashboard hosts
- tablet hosts
- mobile continuity hosts

This allows:
- shared operational systems
- reusable workflows
- portable panes
- specialized standalone experiences
- advanced Connect orchestration

without duplicating entire apps.

---

# Long-Term Direction

The long-term goal is for Pane Packs to become the shared operational language of the EVO ecosystem.

Standalone EVO apps provide focused domain experiences.

Connect provides advanced composition and orchestration.

Pane Packs bridge the two.

The larger idea is simple:

EVO apps should not have to be rebuilt inside Connect.

They should expose the right panes, workflows, and talents, and Connect should compose them.

## Related
- [[EVO Architecture Bible]]
- [[EVO Wiki — One Alice, Many Rooms.md]]
- [[EVO — Adapter Training System.md]]
- [[EVO — Cognition Layer.md]]
- [[EVO — Context Layer.md]]
- [[EVO — Cross-App Context Continuity.md]]
- [[EVO — Global Adapter Distribution Model.md]]
- [[EVO — Shared Embedding System.md]]
^[source-materials/mirrors/doctrine/EVO — Pane Pack Architecture.md]
