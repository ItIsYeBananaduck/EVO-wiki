---
title: alice-hosting-v1-strategy
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/alice-hosting-v1-strategy.md"]
updated: 2026-07-24
---

# EVOconnect Always-On Alice: v1 Deployment Strategy

> Output of [EVOC-327](https://linear.app/lsctech/issue/EVOC-327/05-select-v1-always-on-alice-deployment-strategy): v1 strategy selection for always-on Alice in EVOconnect. Synthesizes findings from [EVOC-324](https://linear.app/lsctech/issue/EVOC-324/01-evaluate-self-hosted-alice-anchor-runtime) (anchor runtime evaluation), [EVOC-325](https://linear.app/lsctech/issue/EVOC-325/02-design-privileged-execution-broker-for-connect) (privileged broker design), [EVOC-326](https://linear.app/lsctech/issue/EVOC-326/03-evaluate-alice-as-user-profile-isolation-model) (OS isolation evaluation), and [EVOC-323](https://linear.app/lsctech/issue/EVOC-323/04-map-governance-and-audit-rules-across-hosting-models) (governance compliance map).

---

## 1. v1 Selected Strategy: Native Desktop Anchor Runtime + Privileged Execution Broker

**Selected model: B + D — Native desktop anchor runtime with the privileged execution broker as its elevation mechanism.**

Alice runs as a user-space background service/daemon launched at login on the user's desktop machine. All elevated actions (filesystem access outside Alice's sandbox, process management, credential reads, and other declared action classes) are delegated to the privileged execution broker defined in [EVOC-325](https://linear.app/lsctech/issue/EVOC-325/02-design-privileged-execution-broker-for-connect). The Delegator state machine governs every task regardless of whether the user is actively present.

This is the unambiguous v1 architecture. No alternative models are selected for v1.

---

## 2. Rationale

### 2.1 Safety

The native desktop anchor satisfies the EVOconnect safety requirements when deployed with mandatory constraints:

- Alice runs as a **non-admin user-space process**. She does not hold elevated OS privileges.
- All elevated operations are routed through the **privileged execution broker**, which independently validates every request against the Delegator's authorization state before executing.
- The broker's action class list is **closed and enumerated** at deployment. No open-ended shell execution is possible.
- Every task must follow the full Delegator lifecycle (`Created → Reviewed → Approved → Executed → Logged → Completed`). Background presence does not grant any bypass of this state machine.
- The anchor holds only **scoped, time-limited ToolGrants**. There are no standing, persistent elevated credentials.
- **Fail-closed**: if the broker or Delegator is unavailable, elevated actions are rejected. Alice does not acquire new capabilities when components are down.
- Tasks are always visible to the user through the Task Manager. Nothing executes invisibly.

The governance compliance map (EVOC-323) confirms that this combined model (B + D) satisfies all six governance principles: Delegator enforcement, full task lifecycle, method binding, auditability, no implicit autonomy, and immutable Talents — provided the mandatory compliance requirements from that document are followed.

### 2.2 Usability

The native desktop anchor directly solves the EVOconnect always-on requirement:

- Alice starts at login and is available without user-initiated launch. This is the primary user value proposition for EVOconnect.
- Talent-authorized background tasks execute without interrupting the user for per-action approval. Method-authorized tasks prompt the user as needed.
- The user retains full control: they can stop the anchor, revoke ToolGrants, and cancel any running task from the Task Manager at any time.
- Audit logs and task history are always inspectable by the primary user.

Standard local runtime (model A) does not satisfy the always-on requirement. Docker self-host (model C) adds operational overhead that degrades usability for the desktop audience without meaningfully improving safety over the native anchor.

### 2.3 Platform Constraints

The native desktop anchor is deployable without infrastructure dependencies:

- **No EVO-hosted cloud infrastructure required.** The anchor runs entirely on the user's machine.
- Targets macOS first (primary EVOconnect platform). The macOS LaunchAgent mechanism provides reliable login-launch, process supervision, and user-space isolation. Equivalent mechanisms exist on Linux (systemd user units) and Windows (Task Scheduler user-context jobs) for future platform support.
- The broker uses per-action OS-level privilege escalation (macOS Authorization Services / polkit on Linux) rather than holding standing admin privileges. This fits within macOS's security model and does not require SIP bypass, full-disk access grants beyond what the workflow needs, or MDM configuration.
- Docker self-host (model C) requires Docker Desktop to be installed and managed by the user, adding a significant runtime dependency and UX burden that is not warranted for v1.

### 2.4 Implementation Complexity

The native anchor + broker combination is the minimum viable architecture that satisfies safety, usability, and platform constraints simultaneously:

- **Anchor**: a persistent user-space process with login-launch configuration, IPC interface for the broker, and connection to the existing Task Manager. No container runtime, no network-layer complexity.
- **Broker**: a narrow, non-agentic gatekeeper with a fixed action class list. Its validation logic is simple (check task state, check ToolGrant, check action class, execute, log). The design is already specified in EVOC-325.
- The broker's scope is intentionally minimal. New action classes can be added in future versions without architectural change.
- Alice-as-user-profile (model E) requires cross-profile IPC, audit log forwarding across OS users, and broker-mediated access for nearly all useful desktop interactions. This integration complexity exceeds what is justified for v1 given that the native anchor already provides the always-on capability.

---

## 3. Deferred Models

### 3.1 Docker Self-Hosted Anchor (Model C) — Deferred

**Reason**: Adds infrastructure and operational complexity without meaningfully improving safety over the native anchor on a desktop machine. Requires Docker Desktop to be installed and managed by the user. Container networking, volume mount management, and ensuring Delegator state and audit logs survive container recreation add implementation burden. The governance compliance map (EVOC-323) requires the Delegator to be active inside the container and audit logs to be persisted to a host-mounted volume — these requirements are satisfiable but non-trivial to enforce reliably in a user-managed deployment.

**Defer condition**: Docker packaging becomes appropriate when a server/NAS deployment use case is validated (headless always-on without a desktop session), or when consistent cross-platform deployment is prioritized over macOS-native integration. Reconsider in the implementation cluster following v1.

### 3.2 Alice-as-User-Profile Isolation (Model E) — Deferred

**Reason**: Provides meaningful blast-radius reduction for desktop automation but requires cross-profile IPC, audit log forwarding from the Alice OS user to the primary user's storage, broker-mediated access for all cross-profile file and UI automation, and profile lifecycle management. The integration complexity is high and the primary safety goal (preventing Alice from running as admin and constraining her tool surface) is already achieved by the native anchor + privileged broker design. The isolation benefit is incremental, not foundational, for v1.

**Defer condition**: Re-evaluate when desktop automation scope expands significantly (e.g., Alice performing sustained multi-app UI workflows on behalf of the user) and the additional isolation boundary provides a commensurate security benefit. The privileged broker design (EVOC-325) must be implemented first — model E depends on it.

### 3.3 Standard Local Runtime Only (Model A) — Excluded

**Reason**: Does not satisfy the always-on requirement. This is not a deployment target for EVOconnect's persistent availability use case. Model A remains the correct mode for on-demand Alice sessions (e.g., a user manually launching Alice for a specific task). It is not deferred — it is simply a different mode that does not address the EVOconnect objective.

**Note**: The standard local runtime mode continues to be supported as the non-anchor interaction mode. It is not replaced by the anchor; it coexists as the user-present, on-demand invocation path.

---

## 4. Mandatory Compliance Requirements for v1

The following requirements are **non-negotiable** for any v1 implementation. They are derived from the governance compliance map (EVOC-323) and the privileged broker design (EVOC-325).

1. **Non-admin anchor process.** The anchor runs as the logged-in user, not as root or an admin account.
2. **Scoped, time-limited ToolGrants.** No standing elevated credentials. Every grant has a TTL and step-count limit.
3. **Full Delegator lifecycle enforcement.** Every task — including background tasks — must traverse the full `Created → Reviewed → Approved → Executed → Logged → Completed` state machine.
4. **No auto-execution of non-Talent tasks.** Background tasks without a valid Talent must not execute without user approval.
5. **Durable, user-inspectable audit logs.** Logs are written before the task is marked `Completed`. The primary user can inspect all task history and broker logs through the Task Manager.
6. **Broker as the only elevated-action path.** Alice does not acquire elevated OS capabilities outside the broker. No direct `sudo`, `osascript` with admin rights, or equivalent.
7. **Cancellable tasks.** The user can cancel any running task and revoke any ToolGrant. The anchor and broker must honor revocation immediately.
8. **Verified method/Talent manifests.** Manifests are hash-verified at load time. A mutable manifest is not trusted.

---

## 5. Follow-on Implementation Issues

The following issues should be created to implement the v1 strategy. They constitute the implementation cluster that EVOC-327 authorizes but does not execute.

### 5.1 Anchor Runtime Implementation

**Proposed issue**: Implement the native desktop anchor runtime for EVOconnect (macOS LaunchAgent).

Scope:
- LaunchAgent plist configuration for login-launch
- Anchor process lifecycle (start, stop, restart, status)
- IPC interface for Alice runtime to connect to the anchor
- Connection to the Task Manager for task visibility
- Health reporting and watchdog behavior

Depends on: EVOC-325 (broker design, already complete).

### 5.2 Privileged Execution Broker Implementation

**Proposed issue**: Implement the privileged execution broker as specified in EVOC-325.

Scope:
- Broker process (non-admin user-space)
- All seven initial action classes (`fs.protected-read`, `fs.protected-write`, `process.manage`, `network.listen`, `credential.read`, `system.query`, `os.notify`)
- Delegator ToolGrant validation integration
- Per-action macOS Authorization Services escalation
- Append-only audit log with fsync-before-return durability guarantee
- Task Manager UI integration for broker log inspection

Depends on: Anchor runtime implementation (5.1).

### 5.3 Task Manager Background Task Visibility

**Proposed issue**: Extend Task Manager to surface anchor-background task history and broker log entries.

Scope:
- UI panel for background tasks executed while the user was away
- Broker log viewer (per-task, filterable by action class and outcome)
- ToolGrant revocation action accessible from Task Manager
- Task cancellation for anchor-running tasks

Depends on: Broker implementation (5.2).

### 5.4 Anchor Governance Integration Tests

**Proposed issue**: Write integration tests verifying governance compliance for the anchor + broker deployment.

Scope:
- Test: non-Talent task does not auto-execute when user is absent
- Test: broker rejects action when ToolGrant is expired
- Test: broker rejects action when task is not in an authorized state
- Test: audit log entry is durable before task transitions to `Completed`
- Test: user ToolGrant revocation propagates to broker within one check cycle
- Test: anchor stops accepting new tasks when broker is unavailable

Depends on: Anchor runtime (5.1) and broker (5.2) implementations.

### 5.5 Docker Anchor Evaluation Revisit (future)

**Proposed issue** (future, not immediate): Evaluate Docker packaging for the anchor runtime for server/NAS deployment use case.

Defer until the server/headless deployment use case is validated. Do not schedule alongside the v1 anchor implementation.

---

## 6. Open Questions Resolved

This section records questions that were open during the cluster analysis phase and are now resolved by this decision.

| Question | Resolution |
|---|---|
| Should Alice run as admin to simplify desktop automation access? | **No.** Non-admin is mandatory. The privileged broker is the only elevated-action path. |
| Is Docker required for the v1 anchor? | **No.** Native LaunchAgent on macOS is sufficient and preferred. Docker is deferred. |
| Should Alice-as-user-profile be implemented in v1? | **No.** Deferred. Complexity exceeds the incremental safety benefit for v1 scope. |
| Can background tasks execute without any Delegator state check? | **No.** All tasks — including Talent-authorized background tasks — traverse the full lifecycle. Talent authorization removes the per-action user prompt, not the state machine. |
| Who owns the escalation path for elevated OS operations? | The privileged execution broker exclusively. Alice has no direct elevated OS access. |
| What happens when the broker is unavailable? | Fail-closed: elevated actions are rejected. Alice continues operating within her non-elevated tool surface. |

---

## Acceptance Criteria Coverage

- [x] **v1 strategy selected and named** (Section 1: Native Desktop Anchor Runtime + Privileged Execution Broker)
- [x] **Deferred models listed with reasons** (Section 3: Docker, Alice-as-user-profile, Standard-only exclusion)
- [x] **Rationale documented covering safety, usability, platform constraints, and implementation complexity** (Section 2)
- [x] **Follow-on implementation issues identified** (Section 5: five proposed issues)

## Related
