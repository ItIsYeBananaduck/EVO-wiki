---
title: "EVOS1-19 Domain Package Design: `mind`, `learn`, `connect`"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-05-12
---

<!-- STATUS: requires-rewrite — contains ENF/VOICE references, must excise before promotion -->

> **Archived — Promoted to Lifecycle System**
> - **Lifecycle stage**: spec
> - **Domain**: connect
> - **Archival date**: 2026-05-12
> - **Archival reason**: Raw note classified and promoted to EVOnotes lifecycle system.
> - **Note**: Canonical/reference copy lives in docs/EVOnotes/spec/connect/. This file is intake history only.

# EVOS1-19 Domain Package Design: `mind`, `learn`, `connect`

_Date_: 2026-03-28  
_Status_: Proposed design (implementation-ready package contracts)

> **Disambiguation:** The `connect` domain package defined in this document is a lowercase AI domain package in the EVO runtime (`packages/domains/connect`). It is **not** EVOconnect, the workspace product. For EVOconnect workspace architecture doctrine, see `docs/EVOnotes/01-core/EVOconnect — Workspace Architecture.md`.

---

## 1) Scope and Goal

This document defines the package/domain design for:

- `packages/domains/mind`
- `packages/domains/learn`
- `packages/domains/connect`

Each domain package must provide:

1. Capability map content
2. Guardrails configuration
3. Executors
4. LoRA mapping strategy

This design assumes shared runtime concerns remain in:

- `packages/ai-runtime`
- `packages/delegator`
- `packages/tools`
- `packages/mesh`

and each domain package only supplies domain overlays/policies.

---

## 2) Common Domain Package Contract

All domain packages should share the same top-level contract shape so orchestration can treat them uniformly.

```text
packages/domains/<domain>/
  pubspec.yaml
  lib/
    evo_<domain>.dart                # public exports
    src/
      capability_map.dart            # domain capability map definitions
      guardrails.dart                # domain guardrail profile + rule IDs
      executors/
        <executor>_executor.dart     # domain executor implementations
        executor_registry.dart       # domain executor registration
      lora/
        lora_profile.dart            # domain LoRA kinds, weights, fallback policy
      domain_binding.dart            # binds map+guardrails+executors+lora profile
  test/
    capability_map_test.dart
    guardrails_test.dart
    lora_profile_test.dart
    executors_test.dart
```

### Standard public interface

Each domain exports a single binding entry point:

- `DomainBinding build<Domain>Binding()`

**Note on runtime contract**: The `DomainBinding` interface defined in `packages/ai-runtime/lib/src/domain_binding.dart` provides the authoritative runtime contract for domain lifecycle (bind/unbind/validate/inspect) and switch event streams. The domain-specific binding factory `build<Domain>Binding()` returns a configured instance of `DomainBinding` (typically `AliceDomainBinding`) with injected domain-specific callbacks for:

- `DomainConfigValidator`: validates domain configuration at bind time
- `DomainOverlayAttacher`: attaches capability map and executor registry during bind
- `DomainOverlayDetacher`: detaches overlays during unbind
- `DomainGuardrailRebinder`: rebinds guardrail profile when switching domains

Each `build<Domain>Binding()` implementation consumes a `DomainBindRequest` (provided as bind-time input) and returns the configured domain binding. The `DomainBindRequest` contains a `DomainConfig` with:

- `domainId` (String): domain identifier (e.g., "mind", "learn", "connect")
- `overlayId` (String): overlay identifier for the capability map version
- `version` (String?): optional version tag
- `guardrailProfile` (String?): optional guardrail profile identifier
- `settings` (Map<String, Object?>): domain-specific settings and metadata

The `build<Domain>Binding()` factory does NOT create `DomainConfig` itself; it receives the configuration via `DomainBindRequest` and uses it to construct the domain binding output with appropriate domain-specific callbacks.

This keeps domain-specific policy composable while the runtime binding interface remains generic and consistent across all domains.

---

## 3) LoRA Adapter Mapping Strategy (Cross-Domain)

### 3.1 Canonical adapter families

Use the existing adapter family naming and extend by domain intent:

**LoRA adapter kinds** (managed via LoRA selection):

- `U`: user-personal adapter (per user)
- `T`: trainer/coach adapter (when role exists)
- `GU`: global user aggregate adapter
- `GT`: global trainer aggregate adapter

**Non-LoRA adapter kinds** (handled by native/runtime layer or separate AdapterChannel):

- `ENF`: always-on enforcement/safety adapter (applied via runtime adapter dispatch, not LoRA family selection)
- `VOICE`: style/voice adapter (optional by surface; applied via runtime adapter dispatch or separate channel)

**Compatibility rule**: `ENF` and `VOICE` are NOT members of `LoRAKind` and MUST be handled outside the LoRA family selection path. LoRA selection operates exclusively on the `U/T/GU/GT` family. `ENF` and `VOICE` adapters are invoked via the runtime adapter dispatch layer or a separate adapter channel, ensuring enforcement and voice styling occur independently of domain-specific LoRA composition.

### 3.2 Domain-specific mapping policy

| Domain               | Primary objective                | LoRA set                                    | Notes                                                            |
| -------------------- | -------------------------------- | ------------------------------------------- | ---------------------------------------------------------------- |
| Training (reference) | Fitness coaching                 | `U + T + GU + GT (+ENF/+VOICE)`             | Existing baseline model                                          |
| Mind                 | Reflective/emotional support     | `U + GU + ENF (+VOICE optional)`            | No trainer concept; emphasize personal longitudinal adaptation   |
| Learn                | Tutoring/pedagogy                | `U + T(educator) + GU + GT(educator) + ENF` | `T/GT` reinterpret as educator expertise adapters                |
| Connect              | Coordination/task/social routing | `U + GU + ENF`                              | Keep lightweight; avoid over-specialized role adapters initially |

### 3.3 Resolution rule

At runtime, adapter selection for a domain should follow:

1. **ENF enforcement layer** (always-on, pre-LoRA): Invoke `ENF` adapter via runtime adapter dispatch before LoRA selection. This ensures safety and guardrail enforcement occurs regardless of LoRA availability.
2. **LoRA family selection** (from `LoRAKind` U/T/GU/GT only):
   a. Load personalized adapter if available: `U`
   b. Load role adapter(s) only if domain defines role authority: `T`
   c. Backfill with aggregate priors: `GU` and optional `GT`
3. **VOICE styling layer** (optional, post-LoRA): Apply optional `VOICE` adapter via runtime adapter dispatch or separate channel based on surface requirements.

**ENF-only fallback**: If all domain-specific LoRA adapters (U/T/GU/GT) are unavailable, the system MUST still invoke the `ENF` adapter and enter deterministic guardrail mode, ensuring enforcement continues even without personalization.

### 3.4 Missing-adapter fallback

- Missing `U` → use `GU`
- Missing `T`/`GT` in `learn` → continue with `U + GU + ENF`
- Missing all domain-specific adapters → `ENF` only + deterministic guardrail mode

---

## 4) Domain Design: `mind`

## 4.1 Capability map sketch

Key capability clusters:

- `mind.reflect`
  - guided journaling
  - pattern reflection and reframing
- `mind.regulate`
  - stress/de-escalation scripts
  - grounding and pause protocols
- `mind.plan`
  - lightweight wellbeing planning
  - routine anchoring and check-ins
- `mind.escalate`
  - crisis deflection and handoff guidance (non-clinical)

### 4.2 Guardrail profile

- **Hard blocks**
  - medical/clinical diagnosis
  - self-harm enabling content
  - legal/financial directives outside scope
- **Constrained outputs**
  - high-emotion cases require short, supportive, non-authoritative responses
  - force “seek qualified help” template for risk signals
- **Evidence policy**
  - no fabricated therapy claims
  - avoid presenting as licensed clinician

### 4.3 Executors

- `mind_reflection_executor`
- `mind_grounding_protocol_executor`
- `mind_checkin_plan_executor`
- `mind_handoff_resource_executor`

### 4.4 LoRA profile

- Default: `ENF + U + GU`
- Optional `VOICE` only in conversational mode
- No `T/GT` by default

---

## 5) Domain Design: `learn`

## 5.1 Capability map sketch

Key capability clusters:

- `learn.explain`
  - concept explanation at selected depth
- `learn.coach`
  - guided practice and hints
- `learn.assess`
  - low-stakes checks and feedback
- `learn.plan`
  - learning path sequencing and milestones

### 5.2 Guardrail profile

- **Hard blocks**
  - unsafe procedural instructions (domain policy dependent)
  - plagiarism/cheating automation requests
- **Constrained outputs**
  - progressive disclosure before full solution dump
  - confidence labels for uncertain content
- **Truthfulness policy**
  - explicit uncertainty and source requirement mode when confidence low

### 5.3 Executors

- `learn_explain_executor`
- `learn_practice_executor`
- `learn_assessment_executor`
- `learn_curriculum_executor`

### 5.4 LoRA profile

- Default: `ENF + U + GU`
- Role-enhanced mode (educator context): add `T + GT`
- `T/GT` are explicitly interpreted as educator adapters (not fitness trainer)

---

## 6) Domain Design: `connect`

## 6.1 Capability map sketch

Key capability clusters:

- `connect.orchestrate`
  - task decomposition and assignment
- `connect.coordinate`
  - communication summaries and action items
- `connect.followup`
  - reminders, status checks, closure tracking
- `connect.integrate`
  - external tool/task-system interaction via approved bridges

### 6.2 Guardrail profile

- **Hard blocks**
  - unauthorized outbound actions
  - data exfiltration or broad bulk-sharing
- **Constrained outputs**
  - require explicit user intent before side-effect actions
  - least-privilege action scope on tool calls
- **Auditability policy**
  - every side-effectful action requires traceable audit entry

### 6.3 Executors

- `connect_task_create_executor`
- `connect_task_route_executor`
- `connect_status_summary_executor`
- `connect_handoff_executor`

### 6.4 LoRA profile

- Default: `ENF + U + GU`
- No `T/GT` in v1 (reassess when role graph matures)
- Prefer deterministic tool-flow behavior over heavy stylistic adaptation

---

## 7) Integration with Shared Runtime

Each domain package should be consumed through `packages/ai-runtime` domain binding APIs:

1. Runtime requests binding for `domainId`
2. Domain provides capability map + guardrail profile
3. Delegator validates action and guardrail constraints
4. Tool registry resolves domain executor
5. Mesh selects LoRA set using domain lora profile

This keeps runtime generic and domain packages policy-focused.

---

## 8) Acceptance Criteria Mapping

- **Domain package structure documented**: Sections 2 and 7
- **LoRA mapping strategy for each domain**: Sections 3–6
- **Guardrails per domain specified**: Sections 4.2, 5.2, 6.2
- **Capability maps sketched**: Sections 4.1, 5.1, 6.1

---

## 9) Recommended Implementation Sequence

1. Scaffold three packages with shared contract files only (no heavy logic)
2. Implement capability maps and guardrail configs as static Dart structures
3. Add executor registry with stub executors and deterministic no-op integration tests
4. Wire LoRA profile resolution into mesh selection path
5. Incrementally replace stubs with domain-specific executor behavior