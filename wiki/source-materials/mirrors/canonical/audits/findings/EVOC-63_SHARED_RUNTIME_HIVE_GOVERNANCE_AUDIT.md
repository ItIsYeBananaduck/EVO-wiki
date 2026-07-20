---
type: audit-finding
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOC-63 Audit: Shared AI Runtime, Hive, Governance, Tools, and Mesh Extraction Map

_Date:_ 2026-03-24  
_Scope audited:_ `flutter_app/`, `packages/`, `apps/evo_connect/`  
_Objective:_ define concrete package boundaries and extraction order for one shared Alice runtime with app-specific domain layers.

---

## 1) Executive Summary

The codebase is in a **hybrid state**:

- Foundational shared packages exist (`packages/core`, `packages/hive`, `packages/sync`, `packages/ui`, `packages/theme`), but only part of runtime/governance/Hive behavior is extracted.
- The **most complete AI runtime and Hive implementations still live in `flutter_app/`**.
- Governance/delegator logic exists in two layers:
  - reusable baseline in `packages/core/src/delegator/*`
  - richer runtime gating + approval surfaces in `flutter_app/lib/features/alice/domain/*`
- Mesh logic exists and is modularized in code shape, but remains partly tied to training-specific semantics.

Primary recommendation: move to a **single shared runtime spine** in `packages/ai-runtime`, `packages/delegator`, `packages/tools`, and `packages/mesh`, while leaving training/mind/learn/connect policy and UX behavior in domain packages.

---

## 2) Proposed Shared Package Migration Map

### Recommended package topology

- `packages/core`
  - common task/log/auth/storage primitives
  - cross-app DTOs and event envelopes
- `packages/ai-runtime`
  - model lifecycle, inference pipeline, session/context assembly, status events
  - runtime adapters and streaming/cancellation contracts
- `packages/hive`
  - node models, registry, state/events, routing contracts
  - discovery/transport abstraction interfaces (platform-specific adapters outside core hive)
- `packages/delegator`
  - tool permission checks, action gating contracts, approvals, audit trails, safe execution boundaries
- `packages/tools`
  - tool registration/invocation/logging/scoping interfaces and standard executors
- `packages/mesh`
  - adapter selection, authority weighting, domain guardrails (generic)
- `packages/domains/training`
  - EVOtraining-specific coaching, mesocycle/planning policies, live workout constraints
- `packages/domains/mind`
  - EVOmind emotional behavior and mind-specific guardrails
- `packages/domains/learn`
  - EVOlearn educational tutoring/routing policies
- `packages/domains/connect`
  - Connect task UI and orchestration shells

---

## 3) Audit Results by Surface

## 3.1 AI Runtime

### What already exists

- Shared runtime-adjacent exports in `packages/core/lib/alice_runtime.dart` and `packages/core/lib/src/alice_runtime/*`.
- Richer runtime orchestration in `flutter_app/lib/features/alice/domain/action_runtime/*` (task lifecycle, validation, execution, persistence flow).
- Platform inference runtime in `flutter_app/ios/Runner/LlamaEngine.swift` and `flutter_app/macos/Runner/LlamaEngine.swift`.
- Runtime presentation adapter in `flutter_app/lib/features/alice/runtime/training_alice_runtime_adapter.dart`.

### Duplicated / fragmented

- Runtime manifest/anchor logic appears in both `flutter_app/lib/features/alice/runtime/*` and `packages/core/lib/src/alice_runtime/*`.
- Execution lifecycle concepts overlap between `packages/core/src/runtime/task_runtime.dart` and `flutter_app` action runtime state machine.

### Classification

- **Shared runtime candidate**
  - `packages/core/lib/src/alice_runtime/*`
  - `flutter_app/lib/features/alice/domain/action_runtime/*` (after API cleanup)
  - cross-platform contracts from `LlamaEngine.swift` (interface extraction first)
- **App-specific wrapper**
  - `training_alice_runtime_adapter.dart` (training avatar/render specifics)
  - training plan/payload conventions embedded in runtime bridge paths
- **Dead code risk / follow-up**
  - duplicate runtime manifest pathways need canonical owner
  - runtime status and state interfaces are split across app/package layers

---

## 3.2 Hive

### What already exists

- Shared baseline package in `packages/hive/lib/src/*`:
  - node model, registry, presence, events, icon system, hive system.
- Production-rich Hive stack in `flutter_app/lib/core/hive/*`:
  - discovery/pairing/state sync/mailbox/transport/work dispatcher/lease manager/inference router/model router.

### Classification

- **Already reusable**
  - `packages/hive/lib/src/hive_node.dart`
  - `packages/hive/lib/src/hive_registry.dart`
  - `packages/hive/lib/src/hive_event.dart`
  - `packages/hive/lib/src/hive_system.dart`
- **UI-coupled**
  - `flutter_app/lib/core/hive/presentation/*`
- **Training-specific or app-coupled**
  - `flutter_app/lib/core/hive/hive_alice_mesh.dart`
  - any Hive path directly binding to training AI choices
- **Safe to extract now**
  - core transport-neutral models + lifecycle orchestration from:
    - `hive_work_dispatcher.dart`
    - `hive_lease_manager.dart`
    - `hive_inference_router.dart`
    - `hive_model_router.dart`
- **Blocked by dependencies**
  - platform plugin ties (`hive_device_plugin.dart`, BLE/network pairing plugins)
  - direct references to app-level registries and UI status surfaces

---

## 3.3 Governance / Delegator

### What already exists

- Reusable baseline in:
  - `packages/core/lib/src/delegator/delegator.dart`
  - `packages/core/lib/src/delegator/safety_rules.dart`
- Rich governance layers in `flutter_app`:
  - `features/alice/domain/gating/*`
  - `features/alice/domain/trainer_approval_service.dart`
  - runtime bridge + execution state machine in `action_runtime/*`

### Classification

- **Shared package candidate (`packages/delegator`)**
  - safety policy engine interfaces
  - approval request/response contracts
  - deterministic allow/block decision API
  - governance audit log schema
- **App-specific**
  - training-only action schemas and fitness-domain restrictions
  - trainer workflow UX/notifications
- **Needs follow-up**
  - reconcile `packages/core` delegator with richer `flutter_app` gating engine into one canonical delegator stack

---

## 3.4 Tools

### What exists today

- Tool-like execution appears as action executors in `flutter_app/lib/features/alice/domain/action_runtime/executors/*`.
- Connect MVP has mock execution agent (`apps/evo_connect/lib/services/codex_agent.dart`) and orchestration (`orchestrator.dart`), but no unified `packages/tools` abstraction.

### Classification

- **Shared candidate (`packages/tools`)**
  - tool registry interface
  - invocation contract + scoped capability checks
  - execution logging contract
- **App-specific**
  - training/business executors (workout plan/body composition specifics)
- **Gap**
  - no unified browser/shell/plugin abstraction package yet; define now before expanding agent actions in multiple apps

---

## 3.5 Mesh / Domain Routing

### What exists today

- Mesh primitives in `flutter_app/lib/features/evolora_mesh/*` (`mesh_engine.dart`, policy/relevance/decision logging).
- Runtime adapter routing bridge in `flutter_app/lib/features/alice/domain/mesh_router.dart`.

### Classification

- **Shared mesh candidate (`packages/mesh`)**
  - weighted selection engine
  - contribution model
  - decision log contracts
- **Domain-specific**
  - mapping rules in `mesh_router.dart` that encode training contexts (`weeklyOverloadDecision`, workout-centric contexts)
  - domain policy defaults tied to fitness modes

---

## 3.6 App-Specific Domain Logic (must stay app-level)

Retain in domain packages (not shared runtime):

- EVOtraining
  - mesocycle services/planners
  - workout/recovery/nutrition/intensity-specific decisioning
  - live workout restrictions and coaching behaviors
- EVOmind
  - emotional response policy and mind-specific intervention patterns (future package target)
- EVOlearn
  - educational sequencing and pedagogy rules (future package target)
- Connect
  - task management shell UX + app-specific orchestration UI

---

## 4) Extraction Matrix

| Component / Surface                        | Current location                                                                                                                         | Target package                                         | Dependency risks                                          | Priority | Difficulty |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ | --------------------------------------------------------- | -------- | ---------- |
| Task runtime + task/log state primitives   | `packages/core/lib/src/runtime/*`, `packages/core/lib/src/models/*`, `packages/core/lib/src/storage/*`                                   | `packages/core` (stabilize, no move)                   | API drift with richer action runtime                      | P0       | Low        |
| Delegator baseline                         | `packages/core/lib/src/delegator/*`                                                                                                      | `packages/delegator` (new)                             | Breaking imports for `evo_core` consumers                 | P0       | Medium     |
| Gating engine + action schema contracts    | `flutter_app/lib/features/alice/domain/gating/*`                                                                                         | `packages/delegator` + domain overlays                 | Fitness-specific rules mixed with generic policy          | P0       | Medium     |
| Action runtime lifecycle engine            | `flutter_app/lib/features/alice/domain/action_runtime/*`                                                                                 | `packages/ai-runtime`                                  | Executor dependencies on training stores/services         | P0       | High       |
| Alice manifest/anchor runtime              | `flutter_app/lib/features/alice/runtime/*` + `packages/core/lib/src/alice_runtime/*`                                                     | `packages/ai-runtime` (single owner)                   | Duplicate implementations, rendering coupling             | P1       | Medium     |
| Native inference contracts                 | `flutter_app/ios/Runner/LlamaEngine.swift`, `flutter_app/macos/Runner/LlamaEngine.swift`                                                 | `packages/ai-runtime` interfaces + platform adapters   | Swift/Flutter bridge coupling, memory constraints         | P0       | High       |
| Hive node/event/registry base              | `packages/hive/lib/src/*`                                                                                                                | `packages/hive` (stabilize)                            | minimal                                                   | P0       | Low        |
| Hive lease/work dispatch/inference routing | `flutter_app/lib/core/hive/hive_lease_manager.dart`, `hive_work_dispatcher.dart`, `hive_inference_router.dart`, `hive_model_router.dart` | `packages/hive`                                        | plugin/device dependencies, app config assumptions        | P1       | High       |
| Hive pairing/discovery plugins             | `flutter_app/lib/core/hive/hive_*pairing*.dart`, `hive_device_plugin.dart`                                                               | `packages/hive` platform adapters                      | BLE/network platform plugins                              | P2       | High       |
| Mesh engine core                           | `flutter_app/lib/features/evolora_mesh/mesh_engine.dart` + supporting models                                                             | `packages/mesh`                                        | naming/context semantics currently training-oriented      | P1       | Medium     |
| Mesh router domain mapping                 | `flutter_app/lib/features/alice/domain/mesh_router.dart`                                                                                 | `packages/domains/training`                            | intentionally app/domain specific                         | P1       | Low        |
| Tool executor contract                     | `flutter_app/lib/features/alice/domain/action_runtime/executors/*`                                                                       | `packages/tools` + domain adapters                     | executor logic mixes business writes and runtime behavior | P1       | Medium     |
| Trainer approval edge-function client      | `flutter_app/lib/features/alice/domain/trainer_approval_service.dart`                                                                    | `packages/delegator` contract + domain-specific client | Supabase function coupling, auth assumptions              | P1       | Medium     |
| Connect orchestration shell                | `apps/evo_connect/lib/services/orchestrator.dart`                                                                                        | `packages/domains/connect` (or remain app-local)       | currently MVP/mock agent assumptions                      | P2       | Low        |

---

## 5) Blocking Dependencies

## AI Runtime blockers

- Runtime engine contracts are not explicitly separated from training payload schemas.
- Native inference behavior is embedded in platform files without shared Dart interface package.
- Duplicate manifest/anchor runtime ownership (`flutter_app` vs `packages/core`).

## Hive blockers

- Discovery/pairing/device plugins are platform-bound and partially UI-driven.
- Core flow references app-level registries and presentation elements.

## Delegator blockers

- Generic safety policy and training-specific rules are interleaved in gating/action schema logic.
- Approval flow depends on concrete Supabase edge functions and app identity assumptions.

## Tools blockers

- No canonical tool registry or invocation protocol package exists yet.
- Executors bundle domain persistence concerns with execution mechanics.

## Mesh blockers

- Context naming and policy defaults are training semantics first.
- Router decisions directly encode workout/planning behavior instead of domain-injected policy modules.

---

## 6) Recommended Extraction Order

1. **Stabilize shared contracts first (P0):**
   - finalize interfaces for runtime events, delegator decisions, tool invocation envelopes, mesh contribution schema.
2. **Split delegator foundation (P0):**
   - create `packages/delegator` from current `packages/core` + selected gating contracts.
3. **Extract AI runtime core (P0/P1):**
   - move action lifecycle engine into `packages/ai-runtime` behind executor interfaces.
   - keep training executors in `packages/domains/training` adapters.
4. **Unify Alice runtime manifest ownership (P1):**
   - one canonical package owner; app-level wrappers only.
5. **Expand Hive extraction (P1/P2):**
   - move lease/work/inference orchestration into `packages/hive`.
   - defer device-plugin transport details behind adapter interfaces.
6. **Extract mesh core (P1):**
   - move generic weighted selection/relevance/logging to `packages/mesh`.
   - keep training context mapping in domain package.
7. **Create tools layer (P1/P2):**
   - define `packages/tools` contracts and migrate shared executor scaffolding.
8. **Finalize domain packages (P2):**
   - explicit app/domain policy overlays for training/mind/learn/connect.

---

## 7) Now vs Later

## Safe to extract now

- Delegator base types + policy engine interfaces
- Runtime lifecycle engine (without training executors)
- Hive core models/events/registry alignment
- Mesh core weighting/relevance/logging primitives

## Extract later (after interface hardening)

- Native inference implementation details
- Platform-specific Hive transport/pairing plugins
- Training-specific routing and policy defaults
- Any UI-coupled Hive/approval/debug presentation layers

---

## 8) Acceptance Criteria Traceability

- AI runtime surfaces audited: ✅
- Hive surfaces audited: ✅
- Governance/delegator/tool/mesh surfaces separated by shared vs app-specific: ✅
- Concrete package map defined: ✅
- Extraction matrix + blockers + ordered plan provided: ✅