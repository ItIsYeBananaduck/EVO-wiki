---
title: "EVOS1-79 — Shared UI Package Architecture"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-08-19
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-79 — Shared UI Package Architecture

This document defines the target shared UI package architecture for EVO and maps the audited Training UI components into package ownership and boundaries.

Reference input:

- `docs/audits/EVOS1-77_TRAINING_UI_SURFACE_AUDIT.md`

## Scope and intent

The goal is to establish a stable package contract before extraction work (EVOS1-80/81/82), so shared UI can evolve without importing Training-specific business logic into shared packages.

## Package architecture (responsibilities and boundaries)

### `packages/design_tokens`

**Ownership**

- Design System / UI Platform (cross-app).

**Included components**

- Canonical design tokens only:
  - Color tokens (brand, semantic, surface, elevation states).
  - Typography scale and font role tokens.
  - Spacing, radii, shadows, border tokens.
  - Motion tokens (durations, easings, choreography constants).
  - Size tokens used by shared shells (card-size variants only once standardized).

**Boundaries (does NOT belong)**

- No rendered widgets/components.
- No app routing/navigation constructs.
- No feature/domain constants (chat prompts, workout-specific labels, role-specific nav items).
- No runtime behavior beyond token definitions.

---

### `packages/ui_core`

**Ownership**

- UI Platform / Shared Components maintainers.

**Included components**

- Cross-app foundational UI primitives:
  - Glass overlays, modal/panel frames, toasts (style/system-level).
  - Core inputs (chat input primitive, text fields, buttons) that are domain-neutral.
  - Generic status indicators/badges.
  - Theme composition adapters that consume `design_tokens`.
  - Layout primitives and reusable shell helpers.

**Boundaries (does NOT belong)**

- No Alice character-specific visuals.
- No chat/business workflows (session management, tool action cards, workout interventions).
- No app-specific screen composition.

---

### `packages/ui_alice`

**Ownership**

- Alice Experience + UI Platform shared ownership.

**Included components**

- Canonical Alice identity and rendering shell:
  - Alice avatar renderer (layered avatar system).
  - Chest-light rendering and anchor contract adapters.
  - Alice glass toast shell (presentation pattern only; composes `ui_core` toasts with Alice-specific glass styling and anchoring behavior; apps should use generic `ui_core` toasts for non-Alice notifications and `ui_alice` toasts when notifications must visually anchor to Alice's character or use Alice-branded presentation).
  - Alice visual state API (idle/talk/listen/attention states) independent of domain data sources.

**Dependencies**

- Depends on `ui_core` for system-level toast primitives (styling and system-level behavior reused in Alice toast shell).

**Boundaries (does NOT belong)**

- No app-specific adapter wiring to Training runtime/state stores.
- No workout/chat business decision logic.
- No app-local copy/content policy.

---

### `packages/ui_chat`

**Ownership**

- Conversational UI maintainers (cross-app) with platform review.

**Included components**

- Conversation UI shells and primitives:
  - Message list/panel shell.
  - Typing and streaming status display primitives.
  - Session-list shell components.
  - Input-to-message layout and affordance patterns.
  - Extensible card slots for app-specific actions.

**Boundaries (does NOT belong)**

- No assistant/runtime orchestration logic.
- No Training-only action-card implementations.
- No app route or navigation ownership.

---

### `packages/ui_projection`

**Ownership**

- Alice/Projection UI maintainers.

**Included components**

- Hologram/projection visual system:
  - Beam + spotlight primitive.
  - Projection card overlay shell.
  - Anchor-to-target projection composition helpers.
  - Projection frame styles and shared presentation tokens.

**Boundaries (does NOT belong)**

- No app-specific payload cards (workout intervention cards, Training chat cards).
- No per-screen placement logic coupled to a specific app route.
- No runtime business triggers for when projection opens/closes.

---

### `packages/ui_motion`

**Ownership**

- UI Platform (animation/motion) with Alice design input.

**Included components**

- Reusable motion choreography and controllers:
  - Alice idle/breath/look/blink/talk motion presets.
  - Projection animation timelines and reusable transition orchestration.
  - Cross-package motion helpers that consume `design_tokens` motion values.

**Boundaries (does NOT belong)**

- No domain event sourcing (e.g., chat backend status engines).
- No app-specific state machines.
- No direct dependency on app-local screens.

## Component → package mapping

The table below maps EVOS1-77 audited components into target package placement.

| Audited component (EVOS1-77)                           | Target package                                                    | Notes                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------ | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Theme system (light/dark)                              | `design_tokens` + `ui_core`                                       | Tokens live in `design_tokens`; ThemeData/composition adapters live in `ui_core`.                                                                                                                                                                                                                                                                                                                                                   |
| Reusable primitive — glass overlays                    | `ui_core`                                                         | Promote current package UI primitive into core package namespace.                                                                                                                                                                                                                                                                                                                                                                   |
| Reusable primitive — chat input field                  | `ui_core` (primitive) + `ui_chat` (shell usage)                   | Decompose existing ChatInputField (`packages/ui/lib/src/chat_input_field.dart`) into a neutral primitive in `ui_core` (remove app-specific defaults like "Ask Alice..." and expose generic callbacks) with a higher-level shell in `ui_chat` that composes the primitive and applies app-specific defaults. See EVOS1-81 for extraction task—implementers should split behavior and defaults rather than only moving the component. |
| Reusable primitive — task/node status badges           | `ui_core`                                                         | Domain-neutral status rendering remains in core.                                                                                                                                                                                                                                                                                                                                                                                    |
| Alice avatar (layered PNG system)                      | `ui_alice`                                                        | Keep renderer and identity visuals shared; app-specific asset/runtime wiring stays local until generalized.                                                                                                                                                                                                                                                                                                                         |
| Alice manifest + anchor system                         | `ui_alice` + `ui_projection`                                      | Manifest/anchor contracts aligned with Alice + projection composition boundary.                                                                                                                                                                                                                                                                                                                                                     |
| Chest light rendering                                  | `ui_alice`                                                        | Canonical character visual primitive.                                                                                                                                                                                                                                                                                                                                                                                               |
| Projection beam + spotlight primitive                  | `ui_projection`                                                   | Core projection painter and effect stack.                                                                                                                                                                                                                                                                                                                                                                                           |
| Projection card overlay shell                          | `ui_projection`                                                   | Shell only; payload content remains app-owned.                                                                                                                                                                                                                                                                                                                                                                                      |
| Projection cards (chat/workout overlays)               | `ui_projection` shell + app-local content                         | Shared container + local payload pattern is required boundary.                                                                                                                                                                                                                                                                                                                                                                      |
| Mobile chat shell                                      | `ui_chat`                                                         | Shared shell with extension slots for app-specific cards and behaviors.                                                                                                                                                                                                                                                                                                                                                             |
| Motion pattern — avatar idle/breathing/look/blink/talk | `ui_motion` (+ consumed by `ui_alice`)                            | Motion definitions in `ui_motion`, bound in Alice renderer.                                                                                                                                                                                                                                                                                                                                                                         |
| Motion pattern — projection animation                  | `ui_motion` (+ consumed by `ui_projection`)                       | Projection timelines owned centrally, consumed by projection components.                                                                                                                                                                                                                                                                                                                                                            |
| Motion pattern — chat typing/status                    | `ui_chat` + `ui_motion`                                           | UI state primitive in `ui_chat`; animation timing from `ui_motion`.                                                                                                                                                                                                                                                                                                                                                                 |
| Reusable primitive — Alice toast                       | `ui_alice` shell (composes `ui_core` toasts) + app-local taxonomy | Alice toast shell composes `ui_core` system-level toast primitives with Alice-specific glass styling and anchoring; use generic `ui_core` toasts for non-Alice notifications; message semantics/triggers remain app-local.                                                                                                                                                                                                          |
| Navigation shell                                       | App-local (outside shared packages)                               | Explicitly excluded from shared UI packages.                                                                                                                                                                                                                                                                                                                                                                                        |
| Card size variant — large/medium/small                 | App-local now, later `design_tokens` when standardized            | Keep local until cross-app token contract is approved.                                                                                                                                                                                                                                                                                                                                                                              |

## Explicit non-goals / exclusions

To protect package boundaries, the following remain out of shared UI packages for this phase:

- Training route graph and bottom-navigation composition.
- Training-specific workout intervention card bodies.
- Training-specific chat business actions and orchestration wiring.
- App role logic (trainer/client routing and tab sets).

## Dependency layering

Target layering for extracted packages:

1. `design_tokens` (no UI package dependencies)
2. `ui_motion` (depends on `design_tokens`)
3. `ui_core` (depends on `design_tokens`; must declare `ui_motion` as a static package dependency in pubspec.yaml, but only the components within `ui_core` that require motion primitives should import and use `ui_motion`—components in `ui_core` that do not need motion must avoid importing `ui_motion` so they remain free of that dependency at the code-import level; build tooling and import rules must allow modules and components in `ui_core` that do not need motion to avoid importing `ui_motion`; this pattern enables selective use of motion choreography from `ui_motion` for transitions, overlays, or toast animations in `ui_core` components that need animation while keeping other `ui_core` components independent of motion primitives)
4. `ui_alice`, `ui_chat`, `ui_projection` (depend on `ui_core` + `ui_motion` + `design_tokens` as needed)
5. Apps (Training and future apps) compose package shells with app-local content and runtime logic

This keeps shared presentation concerns composable while preserving app ownership of domain behavior.