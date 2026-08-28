---
title: "EVOC-102: OpenClaw Reuse / Wrap / Replace / Delete Decision Matrix"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-03-25
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-102: OpenClaw Reuse / Wrap / Replace / Delete Decision Matrix

_Date:_ 2026-03-25  
_Scope:_ OpenClaw → EVOconnect architecture audit artifacts in this repository (`flutter_app/`, `app/`, `apps/evo_connect/`, `packages/*`, native bridges, and operational scripts).  
_Constraint:_ analysis only (no implementation changes proposed in this document).

---

## Decision Legend

- **Reuse as-is**: keep subsystem unchanged and consume directly in EVOconnect.
- **Wrap with adapter**: keep subsystem internals, introduce stable interface/adapters at boundaries.
- **Refactor**: preserve core capability, but restructure internals to remove coupling and technical debt.
- **Replace**: migrate to a new subsystem/implementation because current one is structurally mismatched.
- **Delete**: remove subsystem from migration scope (deprecated, duplicate, or non-strategic).

---

## OpenClaw Subsystem Decision Matrix

| #   | Subsystem                                                              | Current location(s)                                                                                                                      | Decision              | Justification                                                                                                                      | Refactor-phase guidance                                                                                                                |
| --- | ---------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Core task/runtime models                                               | `packages/core/lib/src/runtime/*`, `packages/core/lib/src/models/*`, `packages/core/lib/src/storage/*`                                   | **Reuse as-is**       | Already package-scoped and cross-app oriented; lowest coupling surface.                                                            | Freeze API contracts first, then treat as base dependency for all extracted layers.                                                    |
| 2   | Delegator baseline policy engine                                       | `packages/core/lib/src/delegator/*`                                                                                                      | **Wrap with adapter** | Usable baseline exists, but ownership and API shape should move to dedicated delegator package without immediate behavior rewrite. | Introduce `DelegatorPort` + policy adapter now; postpone policy model changes until governance merge is complete.                      |
| 3   | Rich gating + approval workflows                                       | `flutter_app/lib/features/alice/domain/gating/*`, `flutter_app/lib/features/alice/domain/trainer_approval_service.dart`                  | **Refactor**          | Strong capability but mixed generic policy with training-specific constraints and UI-coupled flows.                                | Split generic approval contract from domain policy packs; isolate Supabase/network clients behind interfaces.                          |
| 4   | Runtime policy + safety-rule primitives                                | `packages/core/lib/src/delegator/safety_rules.dart`, related policy helpers                                                              | **Wrap with adapter** | Rule primitives are reusable but must be consumed through versioned Delegator contracts rather than ad-hoc direct calls.           | Expose policy primitives via stable Delegator-facing interfaces and require contract-version metadata in downstream execution records. |
| 5   | Alice runtime manifest/anchor pathways                                 | `flutter_app/lib/features/alice/runtime/*` + `packages/core/lib/src/alice_runtime/*`                                                     | **Replace**           | Duplicate ownership paths create divergence risk and unclear canonical runtime source.                                             | Consolidate under one canonical runtime package and migrate consumers incrementally via compatibility facade.                          |
| 6   | Native inference bridge contracts (iOS/macOS)                          | `flutter_app/ios/Runner/LlamaEngine.swift`, `flutter_app/macos/Runner/LlamaEngine.swift`                                                 | **Wrap with adapter** | Native engines are required and largely reusable, but direct app coupling is high.                                                 | Define platform-agnostic inference interface and keep per-platform adapters thin and testable.                                         |
| 7   | Hive core node/registry/event model                                    | `packages/hive/lib/src/*`                                                                                                                | **Reuse as-is**       | Already extracted, transport-neutral baseline suitable for shared use.                                                             | Keep stable; prioritize upstream integration tests over redesign.                                                                      |
| 8   | Hive work dispatch/lease/inference routing                             | `flutter_app/lib/core/hive/hive_work_dispatcher.dart`, `hive_lease_manager.dart`, `hive_inference_router.dart`, `hive_model_router.dart` | **Refactor**          | Valuable orchestration logic exists but depends on app configuration and plugin assumptions.                                       | Peel off portable scheduling/routing primitives first, then rebind app-specific dependencies through adapters.                         |
| 9   | Hive pairing/discovery/device plugins                                  | `flutter_app/lib/core/hive/*pairing*.dart`, `hive_device_plugin.dart`                                                                    | **Wrap with adapter** | Platform APIs (BLE/network/device) are inherently environment-specific.                                                            | Standardize plugin contracts and register platform adapters per target shell.                                                          |
| 10  | Tool executor framework                                                | `flutter_app/lib/features/alice/domain/action_runtime/executors/*`                                                                       | **Refactor**          | Current executors mix business-side effects with generic invocation lifecycle.                                                     | Create tool contract registry first; move domain writes behind bounded executor adapters.                                              |
| 11  | Mesh weighted arbitration core                                         | `flutter_app/lib/features/evolora_mesh/mesh_engine.dart` + supporting models                                                             | **Refactor**          | Core arbitration concept is reusable, but semantics are still training-centric.                                                    | Normalize scoring inputs/outputs and move training nomenclature to domain overlays.                                                    |
| 12  | Domain mesh router mapping                                             | `flutter_app/lib/features/alice/domain/mesh_router.dart`                                                                                 | **Wrap with adapter** | Router intentionally encodes domain context; should remain domain-owned but conform to shared mesh interface.                      | Keep domain mappings local; expose only `RouteDecisionProvider` contract upstream.                                                     |
| 13  | Training domain logic (workout/recovery/nutrition/intensity/exercises) | `flutter_app/lib/features/{workout,recovery,nutrition,intensity,exercises}/*`                                                            | **Reuse as-is**       | This is the product-specific domain and should not be generalized into shared runtime packages; scope is `apps/evo_training` only. | Keep in `apps/evo_training` boundary; integrate through stable service interfaces only.                                                |
| 14  | Auth/session/onboarding primitives                                     | `flutter_app/lib/features/auth/*`, portions of shared onboarding flows                                                                   | **Refactor**          | Reusable potential is high, but current flows blend privacy UX, app copy, and orchestration.                                       | Extract auth/session state core first; keep product copy and onboarding narratives app-local.                                          |
| 15  | Chat crypto/local-store/sync substrate                                 | `flutter_app/lib/features/chat/*`, `packages/core/lib/src/chat/*`                                                                        | **Wrap with adapter** | Secure local messaging primitives are reusable; transport implementations differ by surface.                                       | Standardize chat storage/crypto interfaces; implement app-specific transport adapters.                                                 |
| 16  | Wearable + LAN synchronization stack                                   | `flutter_app/lib/features/wearable/*`, `lan/*`, sync managers                                                                            | **Refactor**          | Good cross-app foundation, but naming/state models still reflect legacy training monolith.                                         | Introduce neutral sync domain language and migrate one channel at a time (wearable, then LAN).                                         |
| 17  | SvelteKit marketplace/upload web surface                               | `app/src/routes/marketplace/*`, related web services                                                                                     | **Replace**           | Web app is a parallel product surface, not a reusable foundation for EVOconnect runtime.                                           | Preserve as separate web product; expose integration API contracts instead of sharing internals.                                       |
| 18  | EVOconnect orchestration MVP shell                                     | `apps/evo_connect/lib/services/orchestrator.dart`, `codex_agent.dart`, UI shell files                                                    | **Refactor**          | Useful integration scaffold, but currently mock-heavy and not yet authoritative orchestration layer.                               | Keep as reference consumer while replacing mocks with package-backed runtime contracts.                                                |
| 19  | Duplicate native/watch backup artifacts                                | duplicate watch targets, backup project files under `flutter_app/ios/*`                                                                  | **Delete**            | High maintenance cost and migration confusion; not a strategic subsystem.                                                          | Remove only after canonical targets are validated and release tooling is updated.                                                      |
| 20  | Temporary stubs/sample fixture payloads                                | `app/public/stubs/*`, migration-era placeholders                                                                                         | **Delete**            | Non-authoritative data artifacts can produce drift in runtime behavior and audits.                                                 | Replace with explicit seed pipelines/test fixtures in dedicated test-data locations.                                                   |

---

## Cross-Cutting Guidance for Refactor Phase

1. **Canonical ownership first:** each subsystem above must have one package/app owner before any code motion.
2. **Contract-before-move:** define interfaces/ports before relocating code to avoid coupling regressions.
3. **Domain overlays, not domain leakage:** training/mind/learn/connect policies should implement shared contracts, not mutate core runtime internals.
4. **Compatibility window:** where replace/refactor decisions are selected, keep temporary adapters for staged migration and rollback.
5. **Delete last, not first:** removals should happen only after replacement or adapter-backed flow is validated in the target app shell.

---

## Acceptance Criteria Mapping

- **Every subsystem classified:** complete matrix above with explicit decision per subsystem (1–20).
- **Decisions justified:** each row includes rationale tied to current coupling/reuse profile.
- **Clear guidance for refactor phase:** each row includes actionable refactor-phase guidance, plus cross-cutting migration rules.

---

This matrix is designed to consume outputs from prerequisite architecture audits (`EVOC-101`, `EVOC-103`, `EVOC-106`) as baseline evidence for package boundaries, ownership, and migration sequencing.