---
title: EVO_app_monorepo
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVO_app_monorepo.md
updated: 2026-07-24
---

# EVO System Audit & Extraction Plan

_Date:_ 2026-03-23
_Scope audited:_ `flutter_app/`, `app/`, `apps/`, `packages/`
_Constraint followed:_ analysis only — no code moves or refactors performed.

## Executive Summary

The repository is already in a **partial transition state** from the legacy `EVOtraining` monolith toward a modular monorepo, but the migration is only **structurally started**, not functionally complete.

### What exists today

- `flutter_app/` is still the **largest and most complete implementation** of the product. It contains the real training experience, AI assistant flows, sync logic, local storage, wearable integrations, and most platform-specific code.
- `app/` is a **separate SvelteKit web surface** focused on marketplace/auth/upload workflows. It behaves more like a parallel product surface than a shared package source.
- `apps/evo_connect/` is the **new monorepo app shell**, but it is still an MVP scaffold. It proves package wiring intent, not full extraction completion.
- `packages/` contains the **first-generation shared packages** (`ui`, `theme`, `core`, `sync`), but only `theme` and `ui` are materially usable today. `core` and `sync` are incomplete relative to the monolith.

### Main conclusion

The cleanest path is **not** to migrate whole folders wholesale. Instead:

1. **Stabilize packages first** by extracting the already obvious shared layers.
2. **Keep training-domain logic in `apps/evo_training/`** (new destination for current `flutter_app` domain code).
3. **Treat duplicate/backup/native-experiment folders as cleanup candidates** before or during migration.
4. **Use `apps/evo_connect/` as the consumer/integration app** for package validation, not as the source of truth.

---

## Audit Method

This audit used repository structure review, package manifest review, representative module inspection, and duplicate/placeholder detection. The focus was on:

- shared UI/theme primitives,
- core orchestration/storage/sync utilities,
- AI/helper infrastructure,
- workout/training-specific domain logic,
- deprecated or duplicate implementations,
- extraction blockers caused by native coupling or incomplete packages.

---

## Classification Legend

- **REUSABLE** — should move into shared packages or stay there if already extracted.
- **APP-SPECIFIC** — should remain inside the training app (`apps/evo_training/`).
- **DEPRECATED / DUPLICATE** — should be deleted, archived, or explicitly excluded from migration.

---

# 1. Folder-by-Folder Audit

## A. `flutter_app/`

### What it contains

This is the legacy monolith and still the primary source of production-grade behavior. It includes:

- Flutter app bootstrap, routing, auth, onboarding, and home experience.
- Most business logic for workouts, intensity, nutrition, recovery, trainer workflows, and AI coaching.
- Shared-ish infrastructure such as sync managers, storage abstractions, guardrails, gating, background scheduling, and wearable sync.
- iOS/Android/macOS/Linux/Windows platform projects and native bridges.
- Asset bundles for Alice, models, OCR, and iconography.

### High-level classification

**Mixed**:

- Large reusable infrastructure exists inside this folder and should be extracted.
- The majority of fitness/training behavior remains app-specific.
- Several duplicated or stale native folders/files should not be migrated as-is.

### Recommended sub-classification

#### A1. `flutter_app/lib/core/` — **Mostly REUSABLE**

**Reasoning:** This folder contains monorepo-quality infrastructure rather than training-specific logic.

**Move candidates:**

- Theme primitives and app theming patterns → `packages/theme`
- Background scheduling abstractions and bridges → `packages/core` or `packages/sync`
- Cloud/iCloud sync managers → `packages/sync`
- Shared storage abstractions, widgets, and utility helpers → `packages/core`

**Notes:**

- `flutter_app/lib/core/theme/app_theme.dart` is already effectively duplicated in `packages/theme`.
- Sync code in `core/sync/` is generic enough to be reused across apps, but it still references training-era naming and storage assumptions.

#### A2. `flutter_app/lib/features/alice/` — **Split between REUSABLE and APP-SPECIFIC**

**Reusable portions:**

- guardrails,
- action runtime,
- response gating,
- chat/session infrastructure,
- embedding/vector/federated helper layers,
- autonomy/approval orchestration patterns.

**App-specific portions:**

- coaching flows tied to training plans,
- mesocycle planning,
- body composition and body scan coaching,
- workout-specific Alice flows,
- fitness-domain policy and prompt composition.

**Recommendation:**

- Extract platform-agnostic assistant infrastructure into `packages/core`.
- Keep training-coach logic in `apps/evo_training/`.

#### A3. `flutter_app/lib/features/chat/` — **Mostly REUSABLE**

**Reasoning:** encrypted messaging, local chat store, sync, and report transport are cross-app capabilities.

**Target:**

- crypto/key management/local storage interfaces → `packages/core`
- transport/sync orchestration → `packages/sync`

**Keep app-specific:**

- trainer-client screen composition and workout-report presentation remain in app UI layers.

#### A4. `flutter_app/lib/features/wearable/`, `lan/`, and shared device sync flows — **Mostly REUSABLE**

**Reasoning:** these are platform/system concerns rather than training-only business rules.

**Target:**

- `packages/sync` for cross-device/live-session sync,
- `packages/core` for shared models and abstractions.

#### A5. `flutter_app/lib/features/auth/` and onboarding privacy/sync flows — **Mostly REUSABLE**

**Reasoning:** auth/session/loading/pairing/onboarding consent flows are reusable across EVO apps.

**Target:**

- auth domain models/controllers → `packages/core`
- reusable onboarding UI patterns → `packages/ui`
- sync/privacy onboarding state coordination → `packages/sync`

**Keep app-specific:**

- training-plan onboarding content and fitness preference collection.

#### A6. `flutter_app/lib/features/intensity/`, `workout/`, `recovery/`, `nutrition/`, `exercises/`, most of `home/` — **APP-SPECIFIC**

**Reasoning:** these folders define the product’s training behavior and should remain with the training app.

This includes:

- workout logging,
- intensity scoring,
- strain scoring,
- exercise database behaviors,
- training dashboards,
- readiness/recovery decisions tied to fitness,
- nutrition targets and coaching,
- live workout and workout-plan flows.

**Target:** `apps/evo_training/`

#### A7. `flutter_app/lib/features/evolora_mesh/` — **Mostly APP-SPECIFIC, with selective REUSABLE utilities**

**Reasoning:** the mesh layer is conceptually reusable, but its current implementation is tightly coupled to training/planning decisioning.

**Recommendation:**

- keep current implementation in `apps/evo_training/`,
- later extract only stable abstractions such as decision logs, policy weights, or generic arbitration interfaces after fitness coupling is reduced.

#### A8. `flutter_app/lib/services/` — **Mostly REUSABLE**

Contains top-level services like dashboard metrics and integration adapters that could become shared app services once interfaces are cleaned up.

#### A9. `flutter_app/assets/` — **Mixed**

- Alice brand/avatar assets and generic capability maps are **REUSABLE** across EVO apps.
- Model binaries, OCR data, and training-specific assets should be evaluated case-by-case.
- Example PDFs, draft exports, backups, and temporary placeholders are **DEPRECATED / NON-MIGRATION** candidates.

#### A10. `flutter_app/ios/` and `flutter_app/android/` — **Mixed, high-risk extraction area**

These contain:

- real native integration code that supports app features,
- multiple widget/watch targets,
- duplicate watch directories,
- backup project files,
- testing-only app targets,
- bundled `llama.xcframework` binaries.

**Reusable:**

- generic bridge/plugin layers for AI, sync, widgets, or scheduling.

**App-specific:**

- workout widgets, training-specific live activity, app bundle configuration, training watch scenes.

**Deprecated / duplicate:**

- duplicate watch targets and folder variants,
- backup Xcode project files,
- experimental test targets that overlap canonical ones.

### `flutter_app/` extraction summary

| Area                                                                                         | Classification    | Why                                                              |
| -------------------------------------------------------------------------------------------- | ----------------- | ---------------------------------------------------------------- |
| `lib/core/`                                                                                  | REUSABLE          | infra, theming, storage, background, sync primitives             |
| `lib/features/auth/`                                                                         | REUSABLE-heavy    | shared auth/session/onboarding patterns                          |
| `lib/features/chat/`                                                                         | REUSABLE-heavy    | crypto, sync, local store, transport                             |
| `lib/features/wearable/`, `lan/`                                                             | REUSABLE-heavy    | device/system sync                                               |
| `lib/features/alice/`                                                                        | SPLIT             | assistant infra reusable; training coach logic app-specific      |
| `lib/features/intensity/`, `nutrition/`, `recovery/`, `workout/`, `exercises/`, most `home/` | APP-SPECIFIC      | core fitness domain                                              |
| `assets/`                                                                                    | MIXED             | shared branding vs model/data payloads                           |
| native projects                                                                              | MIXED / HIGH RISK | reusable bridges mixed with training-only targets and duplicates |

---

## B. `app/`

### What it contains

A SvelteKit web application with:

- marketplace pages,
- upload flows,
- auth helpers,
- Supabase client wiring,
- OAuth/music sync dashboard components,
- static onboarding and test/support assets.

### High-level classification

**Mostly APP-SPECIFIC or TRANSITIONAL**, with a few reusable concepts.

### Reasoning

This app is not structured like a shared package source. It is a standalone web surface. Most logic is UI-route-specific, and it does not map cleanly into the existing Dart/Flutter package split.

### Recommended sub-classification

#### B1. `app/src/routes/marketplace/*` — **APP-SPECIFIC**

Marketplace browsing, upload, and program detail flows are product behavior, not shared platform infrastructure.

#### B2. `app/src/components/SignInForm.svelte` and auth wrappers — **Conditionally REUSABLE**

These are reusable only if the monorepo will support multiple Svelte apps sharing UI packages. Otherwise they should stay in the web app.

#### B3. `app/src/lib/supabase.ts`, `stores/auth.ts`, `services/authService.ts` — **REUSABLE ONLY IF web packages are introduced**

These are good candidates for a future `packages/web-core` style package, but **not** for the current Dart package set.

#### B4. `app/public/stubs/*` — **DEPRECATED / DUPLICATE DATA**

These look like temporary JSON fixtures for exercises/foods. They should not be migrated into the formal monorepo unless explicitly adopted as sample/dev seed data.

#### B5. `app/scripts/*` — **Mostly APP-SPECIFIC**, with some cleanup opportunity

Optimization/setup scripts are local to the web app workflow. Backup-generating SVG scripts should not be propagated as shared system architecture.

#### B6. `app/static/onboarding.*` — **APP-SPECIFIC / TRANSITIONAL**

This is a web onboarding micro-app, useful operationally, but not part of the intended Flutter package architecture.

### `app/` extraction summary

| Area                         | Classification              | Why                                              |
| ---------------------------- | --------------------------- | ------------------------------------------------ |
| routes and marketplace flows | APP-SPECIFIC                | web product behavior                             |
| auth/supabase helpers        | CONDITIONAL                 | reusable only if web shared packages are planned |
| public stubs                 | DEPRECATED / NON-MIGRATION  | temporary/sample data                            |
| optimization scripts         | APP-SPECIFIC                | app-local build tooling                          |
| static onboarding            | APP-SPECIFIC / TRANSITIONAL | delivery artifact, not shared package source     |

---

## C. `apps/`

### What it contains

The emerging monorepo app layer. Right now it mainly contains `apps/evo_connect/`.

### High-level classification

**APP-SPECIFIC**, but strategically important as the validation target for shared packages.

---

### C1. `apps/evo_connect/` — **APP-SPECIFIC shell consuming REUSABLE packages**

#### What it contains

- a lightweight Flutter application shell,
- a home screen for task orchestration,
- widgets consuming `evo_ui` and `evo_theme`,
- orchestration services backed by `evo_core`,
- iOS/macOS project shells.

#### Assessment

This folder is a **consumer app**, not the best source for extraction. Its purpose is to prove the modular architecture.

#### Important observations

- It already depends on `packages/theme`, `packages/ui`, `packages/core`, and `packages/sync`.
- The current app proves the package direction is correct.
- However, the package layer is still incomplete; for example, `packages/core` exports model files that are not present yet.

#### Classification

- app shell, features, widgets, and platform files → **APP-SPECIFIC**
- orchestration concepts may eventually belong in shared packages, but current implementations are too MVP-specific to move again immediately.

### `apps/` extraction summary

| Area                          | Classification              | Why                                                              |
| ----------------------------- | --------------------------- | ---------------------------------------------------------------- |
| `apps/evo_connect/` app shell | APP-SPECIFIC                | destination app using shared packages                            |
| its reusable patterns         | Already partially extracted | should be validated against packages, not re-extracted from here |

---

## D. `packages/`

### What it contains

The first pass at the target monorepo package layout:

- `packages/ui`
- `packages/theme`
- `packages/core`
- `packages/sync`

### High-level classification

**REUSABLE by definition**, but maturity differs significantly.

---

### D1. `packages/theme/` — **REUSABLE, GOOD EARLY EXTRACTION**

This is the strongest current package. It already captures shared visual primitives and mirrors the monolith’s theme closely.

**Assessment:** keep and expand.

### D2. `packages/ui/` — **REUSABLE, GOOD EARLY EXTRACTION**

Contains shared visual widgets such as badges, input fields, glass panels, and node indicators.

**Assessment:** keep and expand.

### D3. `packages/core/` — **REUSABLE but INCOMPLETE / BLOCKED**

Contains task/log storage and delegator safety execution, which are exactly the kind of things that belong in a core package.

**Current issue:** package exports reference model files that are not present. That means extraction has started conceptually, but the package is not yet complete enough to become the authoritative source of shared business models.

**Assessment:** high-priority stabilization target.

### D4. `packages/sync/` — **REUSABLE but STILL PLACEHOLDER-LEVEL**

This package has the right responsibility boundary, but current implementation is intentionally minimal and still marked with placeholder/TODO sync behavior.

**Assessment:** expand from `flutter_app/lib/core/sync/` and wearable/device sync layers.

### `packages/` extraction summary

| Package | Classification | State                                    |
| ------- | -------------- | ---------------------------------------- |
| `theme` | REUSABLE       | strongest current package                |
| `ui`    | REUSABLE       | viable and already useful                |
| `core`  | REUSABLE       | incomplete; missing exported model layer |
| `sync`  | REUSABLE       | correct target, but underbuilt           |

---

# 2. Deprecated / Duplicate / Non-Migration Inventory

These items should be **marked for deletion, archival, or explicit exclusion from migration**.

## High-confidence duplicates / stale artifacts

### In `flutter_app/`

- `flutter_app/lib/features/home/presentation/enhanced_music_player_broken.dart`
  Duplicate/broken variant beside the maintained `enhanced_music_player.dart`.
- `flutter_app/ios/WatchApp/` and `flutter_app/ios/watchApp Watch App/`
  Overlapping watch app trees; only one canonical watch target should survive.
- `flutter_app/ios/EvoWidget/` and `flutter_app/ios/EvoFitnessWidget/`
  Overlapping widget/live-activity implementations.
- `flutter_app/ios/Runner.xcodeproj/project.pbxproj.backup`
- `flutter_app/ios/Runner.xcodeproj/project.pbxproj.backup2`
- `flutter_app/ios/Runner.xcodeproj/project.pbxproj.with_watch`
  Backup/generated project variants should not migrate.
- `flutter_app/ios/Runner/alice_capability_map.json.bak`
  Backup file; exclude.
- `flutter_app/assets/DraftsExport-2026-01-27-14-35.txt`
  Working artifact, not a package asset.
- `flutter_app/assets/icons/placeholder.txt`
  Placeholder file.
- duplicate test/demo app targets under `flutter_app/ios/test*`
  likely non-canonical scaffolding.

### In adjacent repo areas relevant to migration hygiene

- `r2-worker/src/index-old.ts`
  old implementation variant.
- root/app-level mixed package manager locks (`package-lock.json` and `pnpm-lock.yaml` in the same app surface)
  not necessarily wrong, but should be normalized before package-oriented migration.
- `app/public/stubs/*`
  likely fixture data, not production shared data.

## Why these matter

If these duplicates are migrated without cleanup, they will:

- create false extraction scope,
- make ownership ambiguous,
- multiply native maintenance cost,
- and confuse which implementation is canonical.

---

# 3. Extraction Plan

## Guiding principle

Extract **stable infrastructure first**, then move the training app into `apps/evo_training/` with the remaining domain code intact.

---

## Package Target: `packages/theme`

### Files / modules to move or consolidate

- `flutter_app/lib/core/theme/*`
- shared color/token helpers currently embedded in feature UIs
- shared typography/spacing constants if they exist outside `core/theme`

### Why

Already partially done. The old monolith theme and new `evo_theme` package overlap almost one-to-one.

### Dependencies

- Flutter Material only.

### Notes

Use `packages/theme` as the canonical source and remove theme duplication from app code after extraction is complete.

---

## Package Target: `packages/ui`

### Files / modules to move or consolidate

- reusable widgets currently in:
  - `flutter_app/lib/core/widgets/*`
  - generic presentation widgets from auth/onboarding/chat/home that do not encode fitness rules
- candidate patterns:
  - status badges,
  - glass panels,
  - reusable input surfaces,
  - reusable cards,
  - onboarding shell components,
  - empty/error/loading states.

### Why

The package already has the correct initial direction. It should become the UI toolkit for both `apps/evo_connect/` and `apps/evo_training/`.

### Dependencies

- `packages/theme`
- Flutter Material

### Do **not** move

- workout screens,
- trainer dashboards with fitness-specific actions,
- nutrition/body-composition widgets tightly tied to training domain state.

---

## Package Target: `packages/core`

### Files / modules to move or consolidate

**High-priority candidates:**

- `flutter_app/lib/features/auth/domain/*`
- reusable parts of `flutter_app/lib/features/alice/domain/`:
  - action runtime,
  - guardrails,
  - gating,
  - conversation/session models,
  - generic autonomy / approval patterns,
  - asset/bootstrap helpers that are not training-specific
- `flutter_app/lib/features/chat/domain/` crypto/key-management/local-store layers
- `flutter_app/lib/features/lan/domain/*` shared models and networking abstractions
- `flutter_app/lib/services/*` where app-agnostic
- selected storage abstractions from `flutter_app/lib/core/`

### Why

This is where the reusable non-UI system logic belongs.

### Dependencies

- local storage packages (`shared_preferences`, `sqflite`, secure storage, etc.)
- crypto packages
- potentially Supabase abstractions, but preferably behind interfaces to avoid package bloat

### Immediate package fix required

Before major extraction, `packages/core` needs:

- actual shared model files added and exported,
- separation between generic models and app-local implementations,
- clear public API boundaries.

---

## Package Target: `packages/sync`

### Files / modules to move or consolidate

- `flutter_app/lib/core/sync/*`
- reusable portions of:
  - `flutter_app/lib/features/chat/domain/chat_sync_service.dart`
  - `flutter_app/lib/features/alice/domain/federated_*`
  - `flutter_app/lib/features/wearable/domain/*`
  - device/local-cloud sync utilities
  - background scheduling and sync status models

### Why

Sync is one of the clearest cross-app concerns in the repo, but it is still scattered across the monolith.

### Dependencies

- `shared_preferences`
- filesystem/path providers
- platform channels or native bridge adapters
- optional network/Supabase clients behind interface boundaries

### Important constraint

Separate **transport/scheduling/state** from **training payload semantics**.
For example:

- generic “sync record”, “job state”, “device sync channel” → package
- workout-specific payload schemas → training app

---

## App Target: `apps/evo_training/`

### Files / modules to keep in the training app

- `flutter_app/lib/features/intensity/*`
- `flutter_app/lib/features/workout/*`
- `flutter_app/lib/features/recovery/*`
- `flutter_app/lib/features/nutrition/*`
- `flutter_app/lib/features/exercises/*`
- most of `flutter_app/lib/features/home/*`
- trainer features tightly tied to fitness planning
- mesocycle/training-plan decision logic
- workout widgets/live activity UX
- training-specific assets and native targets

### Why

This is the actual training product and should remain its own application boundary.

### Dependencies

- `packages/ui`
- `packages/theme`
- `packages/core`
- `packages/sync`
- training-only dependencies that should not leak into shared packages

---

# 4. Dependency & Coupling Assessment

## Strong couplings that increase migration risk

### 1. Flutter feature folders mix domain logic and infrastructure

Many `flutter_app/lib/features/*` directories contain both:

- reusable systems code,
- and training-specific business rules.

**Impact:** extraction cannot be done safely by folder move alone; it must be done by submodule split.

### 2. Native platform code is intertwined with app-specific UX

Widget/watch/live activity code mixes reusable bridge logic with training-specific content.

**Impact:** native extraction needs a “bridge first, target later” split.

### 3. AI/Alice domain code is not cleanly layered yet

Assistant runtime, guardrails, vector memory, federated helpers, and workout-planning logic coexist in the same feature area.

**Impact:** extracting too early without boundary cleanup would create a bloated `core` package that still depends on training concepts.

### 4. New packages are incomplete relative to the monolith

`packages/core` and `packages/sync` are directionally correct but do not yet represent the full shared runtime.

**Impact:** migration should expand packages incrementally, not replace monolith modules all at once.

### 5. Parallel product surfaces (`flutter_app` vs `app`) are not unified by shared packages

The web app uses a different tech stack and packaging model.

**Impact:** do not force-fit web code into the current Flutter package layout. Treat web extraction as a separate packaging stream if needed.

---

# 5. Risk Assessment

## Safe / low-risk areas

- theme tokens and theme objects
- generic UI components
- task/log/delegator primitives
- sync state models
- local-first sync abstractions
- reusable auth domain models/controllers

## Medium-risk areas

- chat crypto and sync
- Alice guardrails and action runtime
- onboarding/privacy flows
- device/wearable sync abstractions
- local storage interfaces used across multiple features

## High-risk areas

- intensity/workout/recovery/nutrition extraction
- Alice planning/mesocycle/fitness intelligence
- native iOS/Android bridge reorganization
- watch app and widget consolidation
- anything involving model assets or on-device AI binaries

## Specific breaking-change risks

- routing/bootstrap assumptions in `flutter_app/main.dart`
- storage schema drift when moving shared stores
- MethodChannel/native bridge names hardcoded in app and native layers
- duplicated native targets making it unclear which bundle/extension is authoritative
- incomplete `packages/core` exports causing build failures if adopted prematurely

---

# 6. Recommended Migration Order

## Phase 1 — Safe extractions

1. Make `packages/theme` the single source of truth.
2. Continue expanding `packages/ui` with generic widgets from the monolith.
3. Repair `packages/core` so exported models actually exist.
4. Move generic task/log/delegator/auth models into `packages/core`.
5. Expand `packages/sync` with generic sync state, local-first persistence, and scheduling primitives.

**Goal:** establish trustworthy package foundations.

---

## Phase 2 — Medium-risk refactors before extraction

6. Split `features/alice/` into:
   - assistant infrastructure,
   - training-domain coaching.
7. Split `features/chat/` into shared transport/crypto vs app UI.
8. Split `core/sync/` and wearable/device sync into generic transport vs training payload logic.
9. Move reusable onboarding/auth flows into package-backed implementations.

**Goal:** reduce cross-cutting coupling before moving more code.

---

## Phase 3 — App boundary creation

10. Create `apps/evo_training/` as the new home for the training app.
11. Move remaining app-specific domain folders from `flutter_app/lib/features/*` into `apps/evo_training/` once shared dependencies point to packages.
12. Keep native training targets with `apps/evo_training/` and only extract reusable bridge code if duplication is proven.

**Goal:** isolate product-specific logic after shared infrastructure is stable.

---

## Phase 4 — Cleanup and deprecation

13. Remove backup, broken, and duplicate files/folders.
14. Choose one canonical watch app tree and one canonical widget implementation.
15. Remove package-manager/file duplication where possible.
16. Archive or delete non-migration fixtures and placeholder assets.

**Goal:** reduce maintenance noise and prevent architecture drift.

---

# 7. Recommended Final Classification Snapshot

## Move to packages

### `packages/ui`

- generic Flutter widgets
- onboarding shell components
- status indicators, badges, panels, inputs, reusable cards

### `packages/theme`

- app theme
- colors/tokens
- global visual constants

### `packages/core`

- auth domain
- task/log/delegator runtime
- reusable Alice runtime helpers
- guardrails/gating/action runtime
- crypto/key-management/storage abstractions

### `packages/sync`

- local/cloud sync abstractions
- background scheduling coordination
- wearable/device sync transport
- federated/sync job state and orchestration primitives

## Keep app-specific in training app

- workout creation/execution
- training plans
- intensity and strain scoring
- nutrition coaching
- recovery logic
- exercise databases and training-specific dashboards
- mesocycle/training decision logic
- training widgets/watch/live activity UX

## Mark for deletion / non-migration

- broken duplicates
- backup project files
- duplicate watch/widget trees
- placeholder assets and fixture data not adopted as canonical
- stale experimental or “old” implementation variants

---

# 8. Final Recommendation

The repository is **ready for extraction planning but not ready for bulk moves**.

The right next step is to:

1. **stabilize shared packages**,
2. **separate reusable infrastructure from fitness-domain logic inside `flutter_app`**,
3. **create `apps/evo_training/` as the explicit destination for app-specific code**,
4. and **clean duplicate native/file artifacts before deeper migration**.

If this order is followed, the monorepo can achieve:

- a durable shared platform layer,
- a clearly isolated training app,
- and much lower long-term migration risk.

## Related

^[source-materials/mirrors/doctrine/EVO_app_monorepo.md]
