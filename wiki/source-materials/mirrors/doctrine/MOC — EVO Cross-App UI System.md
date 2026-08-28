---
title: "Moc — Evo Cross App Ui System"
type: doctrine
tags: ['lsctech', 'doctrine', 'source-material', 'evo']
updated: 2026-08-24
---


## Purpose
Define the shared UI language, reusable surface components, and migration strategy for moving common interface elements into shared packages across the EVO ecosystem.

This work ensures that:
- moving between apps feels consistent
- each app keeps its own purpose and tone
- shared components live in packages instead of app-local duplication
- future apps inherit the same design language by default

---

## North Star

> Moving from app to app should feel like you are in the same house, different room.

This means:
- same visual language
- same interaction philosophy
- same component behaviors
- different layouts and content depending on domain

The goal is not identical screens.

The goal is:
> familiar structure, familiar behavior, different purpose

---

## Core Principle

> Shared shell, domain-specific content.

Every EVO app should share:
- identity
- motion language
- card system
- Alice presence
- chat philosophy
- theme system

But differ in:
- task model
- information density
- primary workflows
- domain-specific content inside components

---

## Shared Cross-App Elements (Confirmed)

### 1. Alice Avatar
Shared across apps as a core identity object.

Characteristics:
- same base Alice presence
- animated through stacked PNG layers
- reusable behavior and rendering model
- app-specific placement or surrounding context allowed

Challenges:
- layered PNG composition
- animation coordination
- sizing across screen contexts
- performance consistency across devices

Decision:
> Alice should become a shared packaged component, not rebuilt per app.

---

### 2. Chest Light Projection Beam
Shared as a core visual identity element.

Characteristics:
- reinforces Alice presence
- creates continuity between chat, overlays, and projection moments
- should be reusable as an effect layer

Decision:
> Projection beam should be packaged as a shared visual effect with configurable intensity, position, and context usage.

---

### 3. Projection Cards
Shared as a reusable surface type.

Characteristics:
- core branded interaction surface
- visually tied to Alice and the EVO identity
- adaptable for domain-specific content

Examples:
- training summaries
- connect focus cards
- mind reflection prompts
- learn lesson surfaces

Decision:
> Projection cards should be part of a shared card family, with reusable shell + pluggable content.

---

### 4. Main Mobile Chat
Shared across apps, at least on mobile.

Characteristics:
- same foundational interaction model
- same Alice-led conversation experience
- domain-specific prompts and context layered on top

Decision:
> Mobile chat surface should be packaged as a shared conversational shell with domain adapters.

---

### 5. Theme System
Shared across all apps.

Includes:
- color palette
- light mode
- dark mode
- glow language
- shadow/elevation language
- status indicators

Decision:
> Theme tokens must be centralized and package-owned.

---

### 6. Card System
Shared across all apps.

Includes:
- large cards
- medium cards
- compact cards
- status dot pattern
- common radius, padding, border, shadow rules

Decision:
> Card shells should be reusable across apps, with only internal content changing by domain.

---

## Shared Design Language

### Identity
- Alice is always Alice
- same color family
- same emotional presence
- same visual intelligence language

### Surfaces
- rounded cards
- projection-inspired overlays
- premium, soft, adaptive depth
- minimal visual noise

### Motion
- smooth, intentional transitions
- no harsh jumps
- state changes should feel alive, not mechanical

### State Indicators
- elegant status dots rather than aggressive full-card state coloring
- subtle, premium, system-wide consistent signaling

---

## What Stays Shared vs What Changes

## Shared
- Alice avatar system
- projection beam
- projection card shell
- card sizing system
- theme tokens
- chat shell
- status dots
- motion principles
- typography rules
- spacing rules
- navigation philosophy

## App-Specific
- content inside cards
- information hierarchy
- domain-specific actions
- task models
- special data visualizations
- app-specific workflows

---

## Product Philosophy

### Same House
Shared across all apps:
- same materials
- same lighting
- same architecture
- same host presence

### Different Room
Per app:
- different furniture
- different tools
- different purpose
- different activity

Examples:
- EVOtraining = fitness room
- EVOconnect = control room
- EVOmind = reflective room
- EVOlearn = learning room

---

## Required Audit

Before moving components into packages, run a UI audit to identify:

### 1. Confirmed Shared Components
What is definitely reused across apps.

### 2. App-Local Components
What should remain domain-specific.

### 3. Component Variants
What is the same component with different content vs what is truly different.

### 4. Technical Constraints
What needs special packaging because of implementation complexity.

---

## Audit Categories

### A. Identity Components
- Alice avatar
- projection beam
- projection effects
- branded interaction elements

### B. Surface Components
- card shells
- overlays
- sheets
- headers
- nav bars
- tab bars

### C. Conversation Components
- chat shell
- message bubbles
- prompt chips
- input bar
- Alice entry point

### D. Theming
- colors
- tokens
- light/dark treatment
- shadows
- borders
- glow rules

### E. Motion
- card expansion
- projection appearance
- Alice idle/active states
- focus shrink/interruption behavior

### F. Domain Adapters
- training-specific content
- connect-specific focus content
- mind-specific reflection content
- learn-specific lesson content

---

## Proposed Package Strategy

### packages/design_tokens
Owns:
- colors
- spacing
- radius
- typography scales
- shadows
- glow tokens
- theme definitions

---

### packages/ui_core
Owns:
- base cards
- buttons
- chips
- headers
- nav elements
- status dots
- layout primitives

---

### packages/ui_alice
Owns:
- Alice avatar renderer
- layered PNG composition
- animation controller
- chest light
- projection beam
- Alice presence widgets

---

### packages/ui_chat
Owns:
- mobile chat shell
- message surfaces
- prompt chips
- input composer
- conversation layout primitives

---

### packages/ui_projection
Owns:
- projection cards
- beam-aware surfaces
- branded overlays
- projection transitions

---

### packages/ui_motion
Owns:
- shared animation timings
- easing rules
- state transition helpers
- focus card adaptive transitions

---

## Migration Strategy

### Phase 1 — Audit
Identify:
- what exists today
- what is reused
- what should be shared
- what must remain local

Output:
- component inventory
- shared vs local classification
- package destination mapping

---

### Phase 2 — Token Extraction
Move:
- colors
- spacing
- typography
- radius
- shadows

into shared design token package first.

Reason:
> everything else depends on this.

---

### Phase 3 — Shared Shell Extraction
Move:
- card shells
- buttons
- chips
- nav elements
- headers

into shared UI core package.

---

### Phase 4 — Alice System Extraction
Move:
- layered avatar rendering
- chest light
- beam projection
- common Alice behaviors

into dedicated shared Alice package.

---

### Phase 5 — Chat Shell Extraction
Move:
- mobile chat shell
- common input surfaces
- common message structure

into shared chat package.

---

### Phase 6 — Domain Recomposition
Rebuild each app using:
- shared shells
- shared themes
- shared Alice presence
- domain-specific content adapters

---

## Important Rule

> Do not package by file similarity alone.

Package by:
- design responsibility
- behavior responsibility
- reuse intent
- ownership boundaries

A component should move only if it is truly:
- cross-app
- stable enough
- conceptually shared

---

## Packaging Test

Before moving any UI element into packages, ask:

1. Is this part of the shared EVO identity?
2. Will at least two apps use it?
3. Is the shell shared even if content differs?
4. Does centralizing it reduce duplication without creating rigidity?
5. Would moving it help the “same house, different room” goal?

If yes:
> move it to packages

If no:
> keep it local for now

---

## Recommended First Audit Deliverable

Create a table with:

- Component name
- Current location
- Shared or local
- Apps using it
- Package destination
- Notes / blockers

Suggested first rows:
- Alice avatar
- chest light beam
- projection card
- mobile chat shell
- card variants
- theme tokens
- status dots
- nav shell

---

## Core Takeaway

> The EVO apps should not feel disconnected.
> They should feel like different rooms in one intelligent environment.

This requires:
- a shared UI language
- a shared component system
- a deliberate packaging strategy
- domain-specific content layered inside reusable shells