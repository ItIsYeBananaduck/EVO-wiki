---
title: EVOtraining — Coach Application Philosophy (Raw Draft)
type: concept
tags: [evo, evotraining, philosophy]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# EVOtraining — Coach Application Philosophy (Raw Draft)

## Purpose

This note defines the operational philosophy and high-level architecture of the EVOtraining Coach experience.

Coach is not simply a trainer dashboard.

It is a coaching-oriented operational environment built around:
- client orchestration
- adaptive programming
- analytics
- AI-assisted coaching
- workflow continuity
- scalable personalization

Coach is designed to allow trainers and coaches to scale their methodology while preserving:
- coaching identity
- coaching philosophy
- strategic oversight
- athlete safety
- human trust

---

# Core Philosophy

Coach is not intended to replace the coach.

Alice exists to amplify the coach’s methodology, patterns, and operational workflows.

Even in high-autonomy modes:
- the coach defines the methodology
- the coach establishes training structure
- the coach controls delegation
- the coach approves strategic direction
- the coach can intervene at any time

Alice operates inside the coach’s approved operational boundaries.

This creates a collaborative coaching system rather than a generic AI fitness platform.

---

# Coach as an Operational Environment

Coach is fundamentally an operational coaching environment.

The primary workflows are:
- client management
- plan construction
- analytics review
- adaptation oversight
- communication
- progression monitoring
- coaching orchestration

The experience should feel:
- smooth
- operational
- lightweight
- contextual
- low-friction
- analytics-oriented

Coach should not feel like:
- spreadsheet software
- enterprise CRM software
- a social platform
- a notification-heavy dashboard

---

# Relationship to EVOconnect

Coach is built using grouped panes and Pane Pack architecture.

The standalone Coach experience acts as a guided host around the Coach Pane Pack.

Inside the standalone Coach app:
- grouped panes are locked
- pane layouts are guided
- workflows are simplified
- navigation is tab-oriented
- advanced composition controls are hidden

Inside Connect:
- the same Pane Pack may be hosted as portable panes
- grouped panes may detach
- pane groups may compose with other workflows
- coaching workflows may integrate with broader operational systems

This allows coaches to begin with a focused standalone experience and later expand into Connect without losing familiarity.

---

# Coach Navigation Philosophy

The standalone Coach app should prioritize simplicity and operational clarity.

Primary navigation should use tabs rather than Connect-style view orchestration.

Potential top-level tabs:
- Dashboard
- Clients
- Calendar
- Plans
- Analytics
- Marketplace
- Billing

The standalone Coach app is intentionally more guided than Connect.

Connect remains the advanced orchestration layer.

---

# Default Coach Workspace

The default Coach operational environment should center around:
- Client Roster
- Calendar
- Alice Coach Chat

The roster acts as the primary operational launcher.

The Calendar is used for:
- in-person sessions
- check-ins
- appointments
- scheduling

The Calendar is not the training-plan structure.

---

# Client Roster Philosophy

The roster is not simply a list of clients.

It is a contextual operational surface.

Client avatars may display lightweight operational indicators.

Examples:
- blue chat bubble = unread client communication
- purple AI insight bubble = Alice generated coaching insight

These indicators should feel:
- lightweight
- ambient
- contextual
- glanceable

rather than noisy or notification-heavy.

---

# Client Workspaces

Clicking a client opens a temporary grouped operational workspace.

This grouped workspace may include:
- Alice Coach Chat
- Client Chat
- Client Plan
- Progress Analytics
- Workout History
- Adherence Trends
- Recovery Trends

The client workspace is temporary by default.

It is not permanently added to the Action Bar.

---

# Session-Scoped Pinning

Client workspaces may be pinned for the current operational session.

Pinning means:
- keep the workspace active during the current session
- preserve the grouped operational context temporarily

Pinning does not mean:
- permanent persistence
- Action Bar registration
- permanent workspace creation

When the session ends:
- temporary client workspaces dissolve
- session pins disappear automatically

This prevents long-term workspace clutter.

---

# Reusable Grouped Workspaces

Coaches may create reusable grouped operational workspaces.

Examples:
- powerlifting team
- bodybuilding prep group
- beginner cohort
- rehabilitation cluster
- nutrition review group

These reusable groups act as operational templates.

Inside Connect:
- groups may be customized
- groups may be composed from multiple client workspaces
- groups may contain shared Alice contexts
- groups may combine analytics across multiple athletes

Example:
- shared Alice pane
- shared group chat
- per-client analytics panes
- shared notes
- comparative progress metrics

This allows Connect to support advanced coaching orchestration workflows.

---

# Coach Pane Philosophy

Coach panes should prioritize:
- analytics
- orchestration
- planning
- review
- communication

rather than workout execution.

Workout execution belongs primarily to:
- mobile
- watch
- athlete-focused flows

Desktop Coach workflows focus on:
- analytics
- adaptation
- oversight
- progression review
- coaching operations

---

# Core Coaching Panes

The Coach Pane Pack may include:
- Client Roster Pane
- Client Profile Pane
- Plan Creation Pane
- Exercise Selection Pane
- Client Analytics Pane
- Progress Trends Pane
- Recovery Trends Pane
- Adherence Pane
- Alice Coach Chat Pane
- Client Messaging Pane
- Marketplace Pane
- Billing Pane
- Review Queue Pane

These panes should remain modular and reusable.

---

# Plan Builder Philosophy

The training plan structure is not calendar-driven.

EVOtraining is designed around:
- adaptive progression
- sequential workouts
- flexible scheduling
- real-world adherence

The core training hierarchy is:

Mesocycle
→ Week
→ Workout
→ Exercises

Users complete workouts on their schedule.

The Calendar remains separate from the training structure itself.

---

# Plan Builder Modes

The Plan Creation Pane should support multiple operational views.

Primary views:
- Mesocycle View
- Week View
- Workout View

Week View should likely be the default operational surface for most coaches.

The plan builder should also support:
- Table Mode
- Drag-and-Drop Mode

Table Mode:
- dense
- spreadsheet-like
- optimized for experienced coaches

Drag-and-Drop Mode:
- visual workflow builder
- exercise dragging
- workout composition
- easier structural editing

The Exercise Selection Pane feeds exercises into the Plan Creation Pane.

---

# Plan History Preservation

Updating a client plan must preserve training history.

Normal updates create:
- new weeks within the current mesocycle

After the deload week:
- the next major update creates a new mesocycle

Prior weeks and mesocycles remain saved.

This allows Alice to reason over:
- prescribed programming
- actual execution
- adaptation response
- adherence
- progression patterns
- historical effectiveness

Coach programming history is append-oriented rather than destructive.

---

# Alice Delegation Philosophy

Alice should earn operational trust over time.

Coach includes a delegation system that controls how much authority Alice has over programming adjustments.

This system is coach-facing only.

Clients do not see delegation controls.

The delegation model exists to:
- preserve coaching identity
- preserve coach authority
- maintain trust
- allow gradual operational adoption

---

# Delegation Levels

Potential delegation levels:

## Insights Only
Alice:
- generates observations
- highlights patterns
- suggests improvements
- identifies concerns

Alice does not modify plans.

---

## Collaborative Mode
Alice may:
- propose plan updates
- suggest modifications
- recommend adaptations

All changes require approval.

---

## Autonomous Weekly Adaptation
Alice may autonomously:
- update weekly programming
- adjust progression
- modify loads/reps/sets
- rotate exercises
- adapt fatigue management

Mesocycle-level structural updates still require approval.

This preserves strategic coaching oversight.

---

# Delegation Scope

Delegation levels should exist:
- globally for the coach
- per-client

The global setting acts as the default operational philosophy.

Per-client delegation allows:
- trust progression
- experimental adoption
- athlete-specific oversight
- prep-client protection
- rehabilitation caution

This allows Alice to gradually earn operational authority.

---

# Alice Communication Philosophy

Most adaptation reasoning should remain conversational.

Alice primarily explains changes through the Alice Coach Chat Pane.

This keeps the system:
- lightweight
- contextual
- operationally fluid

rather than bureaucratic.

Minor adaptations should generally not require heavy approval interfaces.

Major events may generate:
- highlighted messages
- approval prompts
- review requests
- insight badges

---

# Commerce Philosophy

Commerce systems are not fully implemented yet and existing doctrine or code may conflict with this proposed direction.

The intended long-term architecture is:

- coaches define subscription offerings inside Coach
- coaches connect or create Stripe accounts
- EVO orchestrates Stripe setup
- hosted checkout occurs on the EVOtraining website
- clients subscribe through Stripe-powered web flows
- the app manages operational workflows rather than direct Stripe checkout

The app may:
- create offers
- manage offers
- display subscription state
- generate links
- share onboarding flows

The website handles:
- Stripe checkout
- billing
- subscription management
- marketplace purchases

---

# Marketplace Philosophy

The marketplace exists as an operational distribution system for:
- training plans
- coaching programs
- videos
- educational content

Marketplace systems may include:
- recurring listing fees
- hosting/storage fees
- revenue sharing
- Apple-compliant web checkout flows

Marketplace systems should remain separate from the core coaching workflow.

---

# Long-Term Direction

Coach should evolve into a scalable operational coaching environment that allows coaches to:
- scale personalization
- maintain methodological ownership
- leverage AI safely
- orchestrate complex athlete relationships
- integrate with broader Connect workflows

The long-term vision is not to replace coaches.

The vision is to create a collaborative operational environment where Alice becomes an extension of the coach’s methodology.

## Related
- [[EVO Architecture Bible]]
- [[EVOtraining — Adapter Behavior.md]]
- [[EVOtraining — Lab Supplement Intelligence.md]]
^[wiki-native — no upstream source]
