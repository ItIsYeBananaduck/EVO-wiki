---
title: "EVOC-105 — OpenClaw Extension Model vs EVOconnect Plugin Architecture"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-03-25
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-105 — OpenClaw Extension Model vs EVOconnect Plugin Architecture

> Status: Analysis complete (no implementation changes).
> Date: 2026-03-25.
> Scope: Compare OpenClaw extension/plugin model with EVOconnect plugin requirements and identify required EVOfy model changes.

## 1) Scope and evidence

This document compares OpenClaw plugin behavior against EVOconnect plugin requirements:

- capability-based model
- resource exposure model
- permission/governance model
- method/talent support

### Evidence used

#### OpenClaw evidence (current docs, verified 2026-03-25)

- https://docs.openclaw.ai/tools/plugin
- https://docs.openclaw.ai/plugins/architecture
- https://docs.openclaw.ai/plugins/manifest
- https://docs.openclaw.ai/plugins/sdk-overview
- https://docs.openclaw.ai/plugins/building-plugins

#### EVOconnect / EVOfy evidence in this repo

- `docs/evofy/EVOC-106-openclaw-architecture-audit.md`
- `docs/evofy/EVOC-104-openclaw-concept-mapping.md`
- `docs/evofy/EVOC-103-openclaw-governance-conflicts.md`
- `packages/core/lib/src/delegator/delegator.dart`
- `packages/core/lib/src/delegator/safety_rules.dart`
- `packages/core/lib/src/runtime/task_runtime.dart`

---

## 2) OpenClaw extension model (what exists today)

OpenClaw extensions are implemented as plugins with:

1. **Manifest-first discovery** (`openclaw.plugin.json`) and loader validation before runtime activation.
2. **In-process runtime registration** via `register(api)`.
3. **Capability registration APIs** (`registerProvider`, `registerTool`, `registerChannel`, etc.).
4. **Hooks + non-capability surfaces** (commands, services, routes).
5. **Enablement and precedence controls** (`allow`, `deny`, `entries`, `slots`).
6. **A transition model** where capability APIs are preferred, but hook-only/legacy paths remain supported.

Important architectural note: OpenClaw can classify plugin shapes as capability, hybrid capability, hook-only, or non-capability. This means the runtime still accepts extension paths that are not fully capability-contract-bound.

---

## 3) Direct comparison to EVOconnect plugin requirements

### 3.1 Capability-based requirement

### OpenClaw

- Strong capability registration direction exists and is production-used.
- However, hook-only and non-capability plugin shapes remain valid compatibility paths.

### EVOconnect requirement

- Capability boundaries should be explicit and contract-governed for all extension execution paths.

### Gap / conflict

- OpenClaw model is **capability-preferred**, not capability-mandatory.
- EVOconnect requires **capability-mandatory** execution to ensure governance invariants.

### Required EVOfy change

- Define capability contracts as mandatory at runtime boundary.
- Reject (or quarantine) hook-only execution in governed runtime paths.

---

## 3.2 Resource exposure requirement

### OpenClaw

- Plugins can expose many resources/surfaces (tools, channels, routes, services, commands).
- Exposure is powerful but broad; ownership is plugin-centric and shape-dependent.

### EVOconnect requirement

- Resource exposure must be explicit, typed, least-privilege, and auditable by task/action context.

### Gap / conflict

- OpenClaw’s broad SDK surface increases the chance of undeclared or weakly-scoped data/resource access.
- EVOconnect needs consistent resource-scope declarations across all plugin surfaces.

### Required EVOfy change

- Introduce a **resource declaration contract** for every exported plugin surface.
- Add policy-enforced scope classes (e.g., local files, secrets, remote APIs, user-profile domains).
- Deny-by-default any resource path not declared in plugin contract metadata.

---

## 3.3 Permission model requirement

### OpenClaw

- Has plugin allow/deny, enablement rules, and robust tool/sandbox policy patterns.
- Legacy hook pathways can still execute logic outside a strict task-actionability/Delegator gate model.

### EVOconnect requirement

- Delegator-first, deny-by-default governance.
- No silent execution paths.
- Approval lifecycle must be explicit and uniform for high-risk actions.

### Gap / conflict

- OpenClaw permission posture is strong, but not equivalent to a mandatory task-actionability gate.
- Compatibility pathways can weaken deterministic approval/audit behavior.

### Required EVOfy change

- Enforce **Task Actionability Gate** before any plugin-exposed execution.
- Require Delegator approval protocol for privileged/unknown/high-risk actions.
- Block legacy compatibility pathways that bypass Delegator checkpoints.
- Add immutable per-task governance audit records including delegator contract + policy versions.

---

## 3.4 Method/Talent support requirement

### OpenClaw

- Supports rich extension points and automation hooks.
- No native first-class Method/Talent governance semantics equivalent to EVOconnect’s explicit task authorization pathing.

### EVOconnect requirement

- Plugins must interoperate with Method and Talent execution lanes explicitly.
- Every action must carry a resolvable authorization path (method/talent/manual).

### Gap / conflict

- OpenClaw extension metadata does not natively encode Method/Talent execution semantics.
- Authorization path can be implicit rather than structurally enforced.

### Required EVOfy change

- Add plugin contract fields for **method compatibility** and **talent eligibility**.
- Require action invocations to include authorization provenance (`method_id`, `talent_id`, or explicit manual approval id).
- Validate compatibility at dispatch time; fail closed on missing/invalid provenance.

---

## 4) Compatibility decision matrix

| Requirement area               | OpenClaw fit                                           | EVOconnect fit status | EVOfy action                                                            |
| ------------------------------ | ------------------------------------------------------ | --------------------- | ----------------------------------------------------------------------- |
| Capability-based architecture  | Partial (strong direction, non-mandatory)              | Not sufficient as-is  | Make capability contracts mandatory; block hook-only governed execution |
| Resource exposure discipline   | Partial (broad and flexible)                           | Not sufficient as-is  | Add explicit resource declarations + scope enforcement                  |
| Permission/governance model    | Partial (strong controls, weaker actionability parity) | Not sufficient as-is  | Enforce Delegator-first gate + uniform approval + immutable audit       |
| Method/Talent interoperability | Low/Partial                                            | Not sufficient as-is  | Add method/talent contract metadata + dispatch-time provenance checks   |

---

## 5) Required changes to EVOfy plugin model (concrete list)

1. **Capability-contract enforcement layer**
   - Only capability-contract-bound execution is allowed in governed runtime paths.
   - Hook-only extensions are either blocked or isolated to non-governed compatibility sandbox mode.

2. **Plugin contract schema vNext**
   - Add mandatory sections for:
     - declared capabilities
     - declared resources/scopes
     - action risk class + approval requirement hints
     - method/talent compatibility metadata
     - governance contract version compatibility

3. **Delegator gateway integration contract**
   - Every plugin action must pass through a Delegator checkpoint API.
   - No direct plugin-to-tool dispatch without gate evaluation.

4. **Task Actionability Gate binding**
   - Tool exposure and plugin action execution require actionable task state.
   - Explicit deny-by-default when state, approvals, or provenance are missing.

5. **Authorization provenance standard**
   - Standard action metadata envelope includes:
     - `authorization_path` (`method` | `talent` | `manual`)
     - `method_id` / `talent_id` / `approval_id`
     - `delegator_contract_version`
     - `policy_version`

6. **Governance audit contract**
   - Emit immutable per-task/per-action events for attempted and completed calls.
   - Include allow/deny/blocked outcomes with reason codes.

7. **Legacy compatibility policy**
   - Document strict boundary: compatibility support may exist, but cannot run in governed execution lanes.
   - Provide migration path for legacy plugins to capability-contract compliance.

---

## 6) Recommended sequencing (aligned with EVOS1-32 dependency)

1. Finalize tool/package interfaces (EVOS1-32) with capability + governance envelope requirements.
2. Ship EVOfy plugin contract schema vNext.
3. Implement Delegator gateway adapter and Task Actionability Gate at integration boundary.
4. Add audit envelope and versioned governance metadata.
5. Migrate/boundary-wrap legacy plugins; block non-compliant execution paths.

---

## 7) Acceptance criteria check (EVOC-105)

- Clear comparison document: ✅
- List of required changes to EVOfy plugin model: ✅
- No plugin implementation/code refactor performed: ✅