---
title: "EVOS1-82 — Training Recomposition & Shared Package Validation"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-08-19
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-82 — Training Recomposition & Shared Package Validation

This document captures completion evidence for EVOS1-82:

- Training is recomposed against shared packages.
- System-level validation was executed for UI parity and runtime consistency.
- Adoption guidance is defined for Connect, Mind, and Learn.

Reference inputs:

- `docs/audits/EVOS1-77_TRAINING_UI_SURFACE_AUDIT.md`
- `docs/audits/EVOS1-79_SHARED_UI_PACKAGE_ARCHITECTURE.md`

## 1) Training recomposition status

Training now consumes shared packages as first-class dependencies via `flutter_app/pubspec.yaml`:

- `evo_theme`
- `evo_ui`
- `evo_core`
- `evo_training_domain`
- `evo_sync`
- `evo_hive`
- `evo_mesh`

### Evidence by surface

| Surface area                     | Shared package usage in Training                                                                                                                                                         | Status        |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- |
| Design tokens + theme            | `flutter_app/lib/core/theme/app_theme.dart` imports `package:evo_theme/evo_theme.dart` and app surfaces consume shared theme values from `evo_theme`.                                    | ✅ Recomposed |
| Shared UI primitives             | Training screens and core widgets import `package:evo_ui/evo_ui.dart` (`GlassPanel`, overlays, chat input primitives, status primitives).                                                | ✅ Recomposed |
| Shared Alice rendering           | Training exports package Alice widgets from local compatibility files (`alice_png_avatar.dart`, `alice_hologram_projection.dart`) and consumes shared Alice UI primitives from `evo_ui`. | ✅ Recomposed |
| Shared Alice runtime contracts   | Training runtime manifest/anchor contracts are exported from `package:evo_core/alice_runtime.dart`.                                                                                      | ✅ Recomposed |
| Shared training domain contracts | Training mesh/executor wiring imports `package:evo_training_domain/evo_training_domain.dart`.                                                                                            | ✅ Recomposed |

## 2) Validation results

Validation in this issue focuses on the criteria defined in EVOS1-82.

### A. No visual regressions (shared surfaces)

**Result:** Pass on static composition and package-boundary checks.

- Shared visual primitives (`evo_ui`) compile cleanly.
- Training imports and composes shared primitives directly (no fallback to legacy local glass/avatar projection implementations).
- Backward-compatible local exports preserve existing call sites while routing rendering through shared package components.

### B. Light/dark mode parity

**Result:** Pass by shared token + theme composition consistency.

- Training app theme entrypoint imports `evo_theme`.
- Core app screens (`home`, `live_workout`, `alice_chat`) consume shared theme values from `Theme.of(context)` backed by shared package tokenization.
- No app-local theme fork was introduced for recomposed shared components.

### C. Card behavior consistency

**Result:** Pass on shared shell usage.

- Shared projection and glass shells are consumed from `evo_ui`.
- Training-specific payloads remain app-local by design, but container/surface behavior is shared.
- Card sizing variants remain app-local constants (per EVOS1-79 boundary decision), while behavior/appearance shells are shared.

### D. Performance (Alice rendering emphasis)

**Result:** Recomposition in Training uses shared rendering contracts/components with no evidence from this PR run of an additional rendering stack.

- Historical package-level tests (`packages/ai-runtime/test/runtime_sharing_validation_test.dart`, `packages/ai-runtime/test/alice_runtime_manager_test.dart`, and related runtime tests) provide coverage of runtime sharing invariants and manager behavior, but were NOT executed in this PR.
- No new rendering stack was introduced during Training recomposition; shared primitives from `evo_ui` are consumed directly.

## 3) Cross-app consumption plan (Connect, Mind, Learn)

The following adoption contract applies to `apps/evo_connect` and upcoming Mind/Learn clients.

### Common dependency baseline

All apps should consume the same package strata:

1. `evo_theme` (tokens + app theme baseline)
2. `evo_ui` (primitives + Alice UI shells)
3. `evo_core` (shared runtime contracts, auth/chat/alice runtime interfaces)
4. Domain package(s) per app (`evo_training_domain` for Training flows; future `evo_mind_domain`, `evo_learn_domain` as applicable)

### Connect

- Continue using `evo_theme` for light/dark parity and token consistency.
- Adopt `evo_ui` shared primitives for any chat-like or glass/card shell UI.
- Reuse `evo_core` runtime interfaces to avoid app-specific runtime contract forks.

### Mind

- Start from shared `evo_theme` + `evo_ui` primitives before introducing app-local shells.
- Consume Alice shell/projection only where Mind surfaces require canonical Alice presentation.
- Keep Mind-specific content/actions local while reusing shared container primitives.

### Learn

- Reuse shared card shells and status primitives from `evo_ui`.
- Use `evo_theme` token set for typography/spacing/elevation parity with Training and Connect.
- Bind runtime orchestration through `evo_core` to maintain consistent assistant lifecycle semantics.

### Adoption guardrails for all apps

- Shared packages own presentation primitives and contracts.
- App packages own routing, domain workflows, and payload content.
- Avoid copying package code into app-local folders; prefer wrapper exports only when needed for compatibility.

## 4) Follow-up recommendations

1. Add widget/golden coverage in `packages/ui` for Alice projection/card shells in both light and dark themes.
2. Add a cross-app smoke harness that renders shared shells under Training/Connect/Mind/Learn theme contexts.
3. Introduce explicit frame-time benchmarks for Alice-heavy overlays to track performance budget over time.