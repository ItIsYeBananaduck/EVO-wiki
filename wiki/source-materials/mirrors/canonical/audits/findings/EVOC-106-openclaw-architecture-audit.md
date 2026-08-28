---
title: "EVOC-106 — OpenClaw Architecture Audit vs EVOconnect Target"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-03-25
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-106 — OpenClaw Architecture Audit vs EVOconnect Target

> **STATUS: CURRENT** — Last verified against `openclaw/openclaw` `main` tarball downloaded 2026-03-25.
> Review and update this document after any significant change to `packages/core`, `packages/hive`, or the OpenClaw integration boundary.

## 1) Scope and method

This audit inventories OpenClaw architecture and compares it to EVOconnect target constructs:

- control layer
- task delegation
- bounded tools
- plugin system
- Hive
- Swarm
- Delegator

### Evidence used

**OpenClaw evidence** (downloaded source + docs):

- `README.md`
- `docs/concepts/architecture.md`
- `docs/concepts/agent-loop.md`
- `docs/concepts/multi-agent.md`
- `docs/plugins/architecture.md`
- `docs/web/control-ui.md`
- `docs/web/webchat.md`
- `docs/tools/multi-agent-sandbox-tools.md`
- `docs/gateway/sandboxing.md`
- `package.json`
- top-level runtime modules in `src/*`

**EVOconnect target evidence in this repo**:

- `CURRENT_ARCHITECTURE.md`
- `TODO_PENDING.md`
- `packages/core/lib/src/runtime/task_runtime.dart`
- `packages/core/lib/src/delegator/delegator.dart`
- `packages/hive/lib/src/hive_system.dart`

---

## 2) OpenClaw subsystem inventory

## 2.1 Control/runtime plane

1. **Gateway daemon (primary control plane)**
   - Typed WebSocket API, session ownership, channel ownership, event streaming.
   - Serves Control UI and Canvas host surfaces.
2. **Gateway protocol + handshake/auth layer**
   - `connect` handshake, typed request/response/event frames, device pairing, auth modes.
3. **Session subsystem**
   - Gateway-owned session stores/transcripts and lifecycle.

### 2.1 Control/runtime plane

23. **Logging/health/updates/remote access/Tailscale/SSH patterns**

---

## 3) Runtime model, plugin model, task model, UI model, execution model

### Runtime model

OpenClaw uses a **single-host Gateway-centric model** where the gateway is authoritative for sessions, channels, events, and client coordination. Agent runs are embedded and serialized through lane queues.

**Inference:** this is closer to a centralized orchestration runtime than a distributed actor mesh.

### Plugin model

OpenClaw has a **native in-process plugin system** with:

- capability registration (providers/channels/media/speech/image/search),
- hook lifecycle interception,
- manifest-first discovery + validation,
- runtime `register(api)` activation,
- legacy hook-only compatibility path.

### Task model

OpenClaw tasking is primarily **message-driven runs + cron-triggered runs**, not an explicit first-class durable task entity comparable to EVOconnect `ConnectTask`. Session transcripts are the durable execution history primitive.

**Inference:** task semantics are embedded in session/run lifecycle rather than a dedicated task domain object model. This is a fundamental structural difference — OpenClaw has no equivalent to the EVO Task Actionability Gate (tasks non-actionable by default, deny-by-default until Method approved or Talent execution path confirmed).

### UI model

UI is **Gateway-served or Gateway-connected thin clients**:

- Control UI SPA,
- WebChat/native chat clients,
- all using WS gateway methods/events.

### Execution model

Execution is:

- queued per session,
- model-and-tool loop with streaming deltas,
- policy + sandbox mediated,
- optionally plugin-hook intercepted,
- with transcript persistence at gateway.

---

## 4) Comparison against EVOconnect target architecture

## 4.1 Control layer

### OpenClaw

- Very strong, explicit, centralized Gateway control plane.

### EVOconnect target fit

- **Reusable core pattern:** central control plane, typed protocol, client multiplexing, event push.
- **Required adaptation:** map OpenClaw gateway semantics to EVO RuntimeManager/RuntimeContext interfaces and ownership boundaries.

**Disposition:** **REUSABLE (with interface wrappers).**

## 4.2 Task delegation

### OpenClaw

- Delegation emerges through routing/bindings, queue modes, and message tool/actions.
- No explicit Delegator equivalent with built-in user-approval contract in the same shape as EVO `Delegator`.
- No first-class task entity with explicit lifecycle states (Created → Reviewed → Approved → Executed → Logged → Completed).
- No equivalent to the Task Actionability Gate: OpenClaw has no deny-by-default tool access model. Tools are not held hostage until a Method is approved or Talent execution path is confirmed.
- Session transcripts are not a substitute for structured task audit records — they are conversational history, not per-task records with mandatory minimum fields.

### EVOconnect target fit

- EVO requires explicit `ConnectTask` entities with full lifecycle management.
- EVO requires the Delegator Tool Hostage Rule: no tools accessible until the Task Actionability Gate is satisfied.
- EVO requires per-task audit records (Task ID, title, timestamps, authorization path, method/talent reference, tool call summary, outcome, output artifacts).

**Disposition (split):**

- **Task entity domain model → REPLACE** (OpenClaw has no equivalent; build native EVO `ConnectTask` lifecycle from scratch).
- **Run execution pipeline mechanics → REFACTOR** (extract and adapt OpenClaw run-routing/delegation signals into explicit EVO task delegation contracts).

> ⚠️ **Migration constraint:** Any reused run/routing mechanics must be instrumented to emit the Delegator contract version in execution metadata before integration. Delegator contract version must appear in audit logs and inference metadata. If the Delegator contract changes, all tool calls must be revalidated and automatic Talents may require revalidation.

## 4.3 Bounded tools

### OpenClaw

- Strong tool boundaries already exist:
  - policy precedence (`allow`/`deny`/profiles/per-agent/per-provider),
  - sandbox backends,
  - elevated exec carve-outs,
  - per-agent isolation options.

### EVOconnect target fit

- Highly aligned with bounded tool governance goals.

**Disposition:** **REUSABLE** (retain policy concepts, likely wrap config shape and semantics into EVO governance schema).

## 4.4 Plugin system

### OpenClaw

- Mature capability + hooks + runtime registration + SDK.
- Also carries compatibility complexity (legacy hook-only path, very broad SDK surface).

### EVOconnect target fit

- EVO already has a `Delegator` class with full safety-rule validation, user-approval callback lifecycle (including fail-safe deny-by-default), and `approve`/`deny` controls.
- **Wrapper:** keep capability registration pattern.
- **Refactor:** reduce API surface, formalize compatibility tiers, enforce stricter contract boundaries.

> ⚠️ **Safety constraint:** The legacy hook-only compatibility path must be **blocked at the integration boundary**, not just scheduled for deletion. It must never reach the EVO runtime. Any plugin behavior relying on this path can silently bypass the Delegator, violating EVO's no-silent-tool-execution rule. This is a hard safety boundary, not a cleanup task.

**Disposition:** **REFACTOR** (reduce surface, enforce stable contract tiers) + **DELETE** legacy hook-only path (block at integration boundary immediately).

## 4.5 Hive

### OpenClaw

- Multi-agent and node concepts exist, but not as EVO Hive abstraction.
- It has device/node presence + capabilities via gateway/node role and pairing.

### EVOconnect target fit

- Conceptual overlap exists with EVO Hive node registry/presence, but naming/data contracts differ.
- OpenClaw has no equivalent to the EVO Lease Holder — the single active executor responsible for Swarm arbitration, canonical LoRA updates, final Delegator validation, and lease transfer. This must be built independently by the wrapper layer.

**Disposition:** **WRAPPER** (map OpenClaw node presence/caps into EVO Hive model).

> ⚠️ **Gap:** Hive wrapper must implement Lease Holder designation, lease transfer protocol, and offline fallback independently. OpenClaw has no equivalent primitive. Node identity mapping alone is insufficient.

## 4.6 Swarm

### OpenClaw

- No direct evidence of EVO-style Swarm (federated/mesh inference orchestration) as a first-class architectural primitive.
- OpenClaw focuses on assistant runtime/channel orchestration, not federated swarm orchestration.

### EVOconnect target fit

- Current EVO repo marks Swarm/ML areas as still TODO-heavy.

**Disposition:** **REPLACE** (implement EVO Swarm independently; do not attempt direct OpenClaw subsystem transplant).

## 4.7 Delegator

### OpenClaw

- Has tool guards/hook cancellation/sandbox policy, but no first-class Delegator service matching EVO `Delegator` behavior and API.

### EVOconnect target fit

- EVO already has `Delegator` prototype class + safety rules + approval lifecycle.

**Disposition:** **REPLACE (by EVO native Delegator),** while selectively borrowing OpenClaw policy ideas.

---

## 5) Subsystem categorization matrix (required acceptance mapping)

| OpenClaw subsystem                                | EVOconnect category | Rationale                                                                                                                         |
| ------------------------------------------------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Gateway WS control plane                          | **reusable**        | Strong control-plane baseline, aligns with centralized orchestration needs.                                                       |
| Protocol typing + schema/codegen discipline       | **reusable**        | Useful for RuntimeManager/RuntimeContext boundary hardening.                                                                      |
| Session ownership/persistence approach            | **wrapper**         | Reusable concept, but EVO needs task-centric abstractions on top.                                                                 |
| Agent loop pipeline                               | **refactor**        | Keep staged execution structure, convert to explicit task/delegation lifecycle. Must emit Delegator contract version in metadata. |
| Queue/concurrency lanes                           | **reusable**        | Valuable for determinism and race avoidance.                                                                                      |
| Multi-agent routing/bindings                      | **wrapper**         | Adapt routing model to EVO DomainBinding-style interfaces.                                                                        |
| Channel adapter framework                         | **wrapper**         | Useful transport abstraction; re-map to EVO boundaries.                                                                           |
| Node role + device pairing/presence               | **wrapper**         | Good basis for Hive-facing node identity/capability ingestion. Lease Holder semantics must be built independently.                |
| Plugin discovery/validation pipeline              | **reusable**        | Architecture is sound and modular.                                                                                                |
| Capability registration model                     | **reusable**        | Good fit for method/talent style extensibility.                                                                                   |
| Legacy hook-only compatibility path               | **delete**          | Must be blocked at integration boundary — not just scheduled for cleanup. Bypasses Delegator; violates no-silent-execution rule.  |
| Broad plugin SDK subpath surface                  | **refactor**        | Reduce surface and enforce stable contract tiers.                                                                                 |
| Tool policy precedence system                     | **reusable**        | Directly useful for bounded-tool governance.                                                                                      |
| Sandbox backends abstraction                      | **reusable**        | Strong execution boundary primitive.                                                                                              |
| Elevated exec bypass model                        | **refactor**        | Preserve capability but tighten approvals/governance telemetry.                                                                   |
| Control UI/WebChat direct-WS pattern              | **wrapper**         | Useful operator pattern; adapt to EVO surface architecture.                                                                       |
| Cron/job automation                               | **wrapper**         | Reuse for scheduled delegation hooks and orchestrated tasks.                                                                      |
| Task entity domain model (ConnectTask equivalent) | **replace**         | OpenClaw has no first-class task entity. Build EVO task lifecycle natively.                                                       |
| Task audit record system                          | **replace**         | OpenClaw session transcripts do not satisfy EVO minimum audit fields. Build native structured task audit log.                     |
| Swarm-equivalent (not present as core subsystem)  | **replace**         | Build EVO Swarm as dedicated architecture.                                                                                        |
| Delegator-equivalent (not first-class)            | **replace**         | Use EVO Delegator as source of truth.                                                                                             |

---

## 6) Targeted gaps and migration implications

1. **Explicit task domain gap**
   - OpenClaw run/session model is rich but not equivalent to explicit Connect task entities and lifecycle states.
   - The Task Actionability Gate has no OpenClaw analog. Tools are not denied by default; there is no formal "task becomes actionable only after Method approval or Talent execution path" enforcement point. This must be built natively.

2. **Delegator contract gap**
   - OpenClaw has distributed enforcement hooks; EVO expects explicit central task-action governance.
   - Any reused run/routing mechanics must be instrumented to emit Delegator contract version in execution metadata before integration. Contract version must appear in audit logs. When the Delegator contract changes, tool calls must be revalidated and automatic Talents may require revalidation.

3. **Task audit record gap**
   - OpenClaw session transcripts are not structured task audit records. EVO requires per-task minimum fields: Task ID, title, timestamps, authorization path (approved method vs Talent), method/talent reference, tool call log summary, outcome (completed/failed/canceled), and user-visible output artifacts. This must be built natively.

4. **Plugin contract governance gap**
   - OpenClaw's plugin flexibility is high, but EVO should minimize and version hard boundaries.
   - The legacy hook-only path must be blocked at the integration boundary — not deferred as a cleanup task. Any plugin using this path can silently bypass the Delegator.

5. **Hive semantic mismatch + Lease Holder gap**
   - OpenClaw node/pairing primitives can feed Hive, but semantic mapping layer is required.
   - OpenClaw has no Lease Holder concept. The Hive wrapper must implement Lease Holder designation (single active executor for Swarm arbitration, canonical LoRA updates, final Delegator validation), lease transfer protocol, and offline fallback independently.

6. **Swarm absence**
   - No direct subsystem to transplant; EVO Swarm remains a native build concern.

---

## 7) Recommended extraction strategy (no implementation)

1. Extract/control-plane concepts first (Gateway protocol, eventing, session ownership model).
2. Introduce explicit EVO adapters:
   - `RuntimeManager` adapter around gateway orchestration,
   - `DomainBinding` adapter around channel bindings,
   - `RuntimeContext` adapter around run/session context shaping.
3. Keep OpenClaw bounded-tools concepts nearly intact (policy + sandbox), but enforce EVO approval semantics via Delegator. Block legacy hook-only path at this boundary.
4. Build EVO plugin contracts as a narrowed derivative of OpenClaw capability model.
5. Treat Hive as a semantic wrapper over node/pairing primitives — but build Lease Holder, lease transfer, and offline fallback as independent native constructs.
6. Treat Swarm as greenfield EVO subsystem with selective utility reuse only.
7. Build EVO `ConnectTask` entity lifecycle and structured task audit log natively — do not attempt to derive these from OpenClaw session transcripts.

---

## 8) Acceptance criteria check

- **All major subsystems identified:** ✅
- **Each subsystem categorized (reusable/wrapper/refactor/replace/delete):** ✅
- **Clear comparison against Connect architecture (control layer, task delegation, bounded tools, plugin system, Hive, Swarm, Delegator):** ✅
- **Task entity domain gap and Delegator contract versioning requirement flagged:** ✅
- **Legacy hook-only path safety boundary called out:** ✅
- **Lease Holder gap documented:** ✅
- **Task audit record gap documented:** ✅