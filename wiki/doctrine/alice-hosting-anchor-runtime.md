---
title: alice-hosting-anchor-runtime
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/alice-hosting-anchor-runtime.md
updated: 2026-07-24
---

# EVOconnect Self-Hosted Alice Anchor Runtime

> Output of [EVOC-324](https://linear.app/lsctech/issue/EVOC-324/01-evaluate-self-hosted-alice-anchor-runtime): evaluation of the desktop-first self-hosted Alice runtime acting as a persistent Hive anchor node.

This document is referenced by [EVOC-325](https://linear.app/lsctech/issue/EVOC-325/02-design-privileged-execution-broker-for-connect) (privileged broker design) and [EVOC-327](https://linear.app/lsctech/issue/EVOC-327/05-select-v1-always-on-alice-deployment-strategy) (v1 strategy selection). It does not implement the runtime; it defines the runtime's responsibilities, boundaries, and risks.

---

## 1. Runtime responsibilities

The anchor runtime is a persistent user-space process that runs on the user's desktop machine. It is the local node that keeps Alice available when the user is not actively interacting with an EVOconnect application. It is not a replacement for the standard local runtime; it is the always-on mode.

**Core responsibilities:**

1. **Maintain Alice presence** — start at user login, keep the Alice runtime alive, and restart it if it crashes or exits unexpectedly.
2. **Host the local Hive node** — act as the authoritative on-device coordinator for Hive protocol traffic, task scheduling, and state synchronization.
3. **Coordinate with the Task Manager** — receive tasks, report lifecycle state, and make task history visible regardless of whether the user was present when the task ran.
4. **Route elevated actions to the broker** — never execute privileged OS operations directly. Forward all elevated action requests to the privileged execution broker defined in [EVOC-325](https://linear.app/lsctech/issue/EVOC-325/02-design-privileged-execution-broker-for-connect).
5. **Hold scoped, time-limited ToolGrants** — obtain grants from the Delegator and refresh or drop them before they expire. The anchor does not store standing credentials.
6. **Persist minimal durable state** — keep enough state (task queue, in-progress task status, pending approvals) to survive a restart without losing user-visible work or audit records.
7. **Expose health and status** — report whether the anchor, Alice runtime, and broker are running, and surface degradations through the Task Manager.

**What the anchor is not:**

- Not an admin or root process. It runs as the logged-in user.
- Not an autonomous agent. It does not decide what tasks to run; it hosts the runtime that executes tasks approved by the Delegator.
- Not a replacement for the privileged broker. It does not perform elevated actions itself.
- Not a cloud service. It does not rely on EVO-hosted infrastructure and does not send task data to EVO servers unless an explicit user-authorized integration requires it.

---

## 2. Docker vs. native background service

Two packaging/deployment options exist for the anchor runtime: a native platform background service (LaunchAgent on macOS, systemd user unit on Linux, user-context Task Scheduler job on Windows) and a user-managed Docker container. Both satisfy the "self-hosted" requirement, but they differ in security semantics, operational burden, and platform fit.

### 2.1 Native desktop background service (v1 preferred)

The anchor runs as a user-space process managed by the OS login session. On macOS this is a `LaunchAgent` in the user's domain; on Linux, a `systemd --user` unit; on Windows, a user-context scheduled task.

**Pros:**

- No extra runtime dependency beyond the OS. The user does not need to install or manage Docker Desktop.
- Tight integration with the user's session: keychain access, notifications, UI automation, and filesystem permissions all work under the logged-in user's identity.
- Lightweight footprint appropriate for a desktop machine that is not always on or always connected.
- Aligns with the EVOconnect on-device-first principle: Alice runs on the user's hardware with minimal abstraction layers.

**Cons:**

- Platform-specific packaging (one plist/unit/job per platform).
- Less portable than a container image across different OS distributions.
- The OS provides the isolation boundary; a bug in the anchor has the same reach as any other user-space process running under the user's identity.

**Security role:** The native process runs as the user. Its security boundary is the OS user identity plus the privileged broker for any operation that requires more than the user's normal rights. The anchor itself is not a sandbox.

### 2.2 Docker self-hosted anchor

The anchor runs inside a Docker container on the user's machine. The container is built from an image that contains the Alice runtime, anchor supervisor, and verified method/Talent manifests.

**Pros:**

- Single packaging artifact for multiple platforms (where Docker is available).
- Easier to reproduce across machines and share exact image hashes for verification.
- Container image can be signed and hash-verified, giving a clear deployment artifact.

**Cons:**

- Requires Docker Desktop or an equivalent engine to be installed and kept running by the user.
- Desktop integration is weaker: keychain access, OS notifications, UI automation, and session-bound permissions require extra host bindings that erode the container boundary.
- Volume mounts and networking must be explicitly configured; misconfiguration (mounting the whole home directory, passing the Docker socket) can give the container more access than the native anchor would have.
- On macOS, Docker runs in a VM, adding memory and CPU overhead without improving security for the desktop use case.

**Security role:** Docker is a **packaging and deployment convenience**, not a security boundary. On a desktop machine, the container usually runs with the same effective reach as a native user-space process once the required host mounts are granted. The privileged broker remains the only elevation path. Running the container as root, with `--privileged`, or with the Docker socket mounted turns the model into a high-risk deployment and is explicitly non-compliant with the governance map (EVOC-323).

### 2.3 Comparison summary

| Dimension | Native background service | Docker self-hosted anchor |
|---|---|---|
| Dependency | OS-native launch mechanism | Docker Desktop / container engine |
| Desktop integration | Native | Requires host mounts and IPC plumbing |
| Packaging | Platform-specific | Cross-platform image |
| Runtime overhead | Minimal | VM overhead on macOS |
| Security boundary | OS user identity + broker | Same effective reach once mounts are granted; Docker is not a sandbox here |
| Operational burden | Low | Higher (image updates, volume management, engine health) |
| v1 fit | Preferred | Deferred |

**Conclusion for v1:** Select the native desktop background service. It satisfies the always-on requirement with the lowest operational burden and the cleanest desktop integration. Docker is deferred to a future server/NAS or cross-platform packaging use case. This matches the v1 strategy selection in [EVOC-327](https://linear.app/lsctech/issue/EVOC-327/05-select-v1-always-on-alice-deployment-strategy).

---

## 3. Resource access model

The anchor runtime needs access to resources on the user's machine. The resource model distinguishes between resources the anchor owns, resources it accesses on behalf of the user, and resources that require broker-mediated elevation.

### 3.1 Anchor-owned resources

These are files, sockets, and state stores that the anchor creates and manages itself:

| Resource | Purpose | Location guidance |
|---|---|---|
| Anchor working directory | Task queue, health snapshots, temporary runtime state | User-owned directory, e.g., `~/Library/Application Support/EVOconnect/anchor` on macOS; platform-specific elsewhere |
| Audit log stream | Forwarded task/broker log entries pending Task Manager sync | Same working directory, written with append-only semantics |
| IPC sockets | Communication between Alice runtime, anchor, and broker | Restricted to user session (e.g., Unix domain socket in working directory) |
| Process identity | The anchor's own process table entry and child processes | Runs under the logged-in user's UID |

These resources should never be shared with another OS user or exposed over the network.

### 3.2 User resources accessed directly

The anchor and the Alice runtime it hosts may access files and data that belong to the user, within the user's normal permissions. This is the standard, non-elevated access model:

| Resource class | Examples | Access rule |
|---|---|---|
| User documents | Notes, projects, downloads | Read/write according to the user's file permissions |
| User configuration | Application preferences, EVOconnect settings | Read/write within the user's home directory |
| Local integrations | EVOsync, local plugins, on-device models | Access only paths the user has explicitly authorized |
| Plugins and Talents | Verified Talent manifests, plugin bundles | Load from user-authorized directories; verify hash before trusting |

The anchor must not silently expand its access beyond what the user or an authorized Talent has declared. It must not scan the entire filesystem unless a specific, authorized task requires it and the scope is recorded in the task log.

### 3.3 Elevated resources accessed through the broker

Any resource outside the user's normal permission scope is accessed through the privileged execution broker. The broker validates each request against the Delegator's authorization state. Examples include:

| Resource class | Examples | Broker action class |
|---|---|---|
| Protected files | System config files, another user's data | `fs.protected-read`, `fs.protected-write` |
| System processes | Restarting a user-space background service | `process.manage` |
| Network listeners | Binding a local port for a dev server | `network.listen` |
| Credentials | Reading an API key from the system keychain | `credential.read` |
| System state | Disk usage, installed packages | `system.query` |

The anchor must never attempt to bypass the broker by using `sudo`, `osascript` with admin rights, or equivalent mechanisms. The broker is the exclusive elevation path.

### 3.4 Resource access boundaries

These boundaries apply regardless of whether the anchor is native or Docker-hosted:

1. **Non-admin identity.** The anchor process runs as the logged-in user, not as root or an admin account.
2. **No host-wide mounts in Docker.** If Docker is used, only specific workflow directories are mounted. Mounting the entire host filesystem, `/var/run/docker.sock`, or giving `--privileged` is prohibited.
3. **Plugin/Talent hash verification.** Plugins and Talents are loaded only from declared directories and only after their manifest hashes are verified.
4. **No network exposure.** Anchor IPC sockets and coordination endpoints are local to the device. No inbound network ports are opened by the anchor unless a user-authorized task explicitly requests it through the broker.
5. **Credential non-retention.** The anchor does not store credentials. It may request a credential read through the broker, but the broker returns the credential to the task context, not to the anchor's persistent state.

---

## 4. Hive coordination role and responsibilities

In the EVO runtime architecture, the Hive protocol coordinates Alice across multiple devices and processes. The anchor runtime is the persistent on-device Hive node for the user's desktop.

### 4.1 Anchor as Hive node

The anchor fulfills the following Hive roles:

- **Persistent presence.** The anchor keeps a Hive node identity online for the local device even when no foreground EVOconnect app is running. This is the foundation for background tasks and asynchronous task completion.
- **Task ingress.** It receives task requests from other devices (via Hive sync) and from local triggers, and submits them to the local Task Manager/Delegator.
- **State synchronization.** It syncs task state, audit events, and metadata with the user's EVOsync-backed storage zones so that other devices see progress and history.
- **Swarm coordination.** When a task requires parallel inference (Swarm), the anchor acts as the local coordinator. It distributes shard work tickets to available Swarm nodes and merges results. Swarm nodes do not invoke the privileged broker directly; only the anchor may request broker actions on behalf of the local task.

### 4.2 Relationship to the Task Manager and Delegator

The anchor is a runtime host, not the source of truth for task authorization:

- The **Task Manager** owns task history, user-visible task state, and cancellation/ revocation controls. The anchor reports into it.
- The **Delegator** owns the task authorization state machine (`Created → Reviewed → Approved → Executed → Logged → Completed`). The anchor cannot execute a task unless the Delegator has placed it in an authorized state with a valid `ToolGrant`.
- The **privileged broker** owns elevated OS action execution. The anchor routes requests to it.

The anchor's job is to keep the local runtime ready so that authorized tasks can execute promptly. It does not relax or shortcut the Delegator's lifecycle.

### 4.3 Background task rules

Because the anchor runs while the user may be away, these rules are mandatory:

- **Talent-only auto-execution.** A task may execute without a per-action user prompt only if it is bound to a valid, immutable Talent that declares the required tools. Method-bound tasks still require user approval.
- **No silent start.** The user must know the anchor is running. The anchor registers with the OS session and appears in the Task Manager's device/node list.
- **Cancellable at any time.** Any task running under the anchor can be cancelled, and any `ToolGrant` can be revoked, from the Task Manager.
- **Audit before completion.** The task cannot transition to `Completed` until its audit log entry is durable. The anchor forwards the log entry to the Task Manager and/or broker log store before signaling completion.

---

## 5. Task orchestration responsibilities

The anchor is responsible for executing the local lifecycle of tasks that have been authorized by the Delegator. It is not responsible for deciding what tasks should run.

### 5.1 Task lifecycle operations

| Phase | Anchor responsibility |
|---|---|
| Created | Receive task from Hive/Task Manager; stage it in the local queue |
| Reviewed | No action; wait for Delegator/approval flow |
| Approved | Verify valid `ToolGrant`; prepare runtime context (plugins, models, sandbox) |
| Executed | Run the task in the Alice runtime; route elevated actions to the broker |
| Logged | Forward task audit records to the Task Manager/broker log store |
| Completed | Acknowledge completion; release task-scoped resources and grants |

### 5.2 Runtime context preparation

When a task is approved, the anchor prepares a minimal execution context:

- Load the method or Talent manifest and verify its hash.
- Resolve the declared `ToolGrant` and confirm it is not expired and covers the requested tools/action classes.
- Mount or bind only the user resources declared in the task scope.
- Start the Alice runtime process if it is not already running, or attach the task to the existing runtime.

### 5.3 Concurrency and isolation

- Multiple tasks may be queued but each task executes within its own runtime context. The Alice runtime may multiplex tasks, but the anchor ensures each task's grants and scope are isolated.
- The anchor does not share one task's `ToolGrant` with another task.
- If the Alice runtime crashes, the anchor restarts it and resumes queued tasks from durable state, not the in-memory state of the crashed process.

### 5.4 Swarm orchestration

When the anchor receives a Swarm-sharded task:

1. It holds the parent task context and `ToolGrant`.
2. It creates work tickets for each shard and dispatches them to available local Swarm nodes.
3. It collects shard results and applies the Swarm merge rule.
4. If a shard requires an elevated action, the anchor submits the broker request on behalf of the parent task. Swarm nodes never submit broker requests directly.
5. It reports the merged result and task state to the Task Manager.

---

## 6. Runtime constraints and risks

### 6.1 Constraints

| Constraint | Requirement |
|---|---|
| Non-admin execution | The anchor must run as the logged-in user, not root or admin |
| Broker-only elevation | All privileged OS actions go through the privileged broker |
| Delegator enforcement | No task executes without a valid `ToolGrant` and authorized state |
| Scoped grants | ToolGrants are time-limited and step-limited; no standing grants |
| Durable audit logs | Task and broker logs must be written before completion |
| Local-only IPC | Anchor IPC is local to the device; no inbound network listeners |
| User control | User can stop, restart, and inspect the anchor at any time |
| Immutable Talents | Talent manifests are hash-verified and immutable |

### 6.2 Risks

| Risk | Severity | Mitigation |
|---|---|---|
| Anchor compromise grants attacker persistent access to the user's machine | High | Run as non-admin user; broker-only elevation; scoped, expiring ToolGrants; user can stop/revoke; audit logs visible in Task Manager |
| Background task runs without user approval while user is away | Medium | Auto-execute only Talent-bound tasks; method-bound tasks require user confirmation; all tasks visible in Task Manager |
| Docker misconfiguration gives container broad host access | High | For Docker: non-root container, no `--privileged`, no Docker socket, limited host mounts, state/logs on host; prefer native service for v1 |
| Anchor process crash loses in-progress task state | Medium | Persist task queue and state to anchor working directory; resume from durable state after restart |
| Anchor logs or state are tampered with by malware running as the same user | Medium | Append-only logs; broker log store separate from Alice logs; user inspects logs through Task Manager; regular sync to EVOsync-backed zone |
| Plugin/Talent with malicious manifest is loaded | Medium | Hash-verify manifests; load only from user-authorized directories; Talents are immutable snapshots |
| Resource exhaustion (CPU, memory, disk) from long-running background tasks | Medium | Task-level limits, watchdog, user cancellation, and health reporting through Task Manager |
| Cross-device task spoofing via Hive sync | Medium | Tasks are validated by the Delegator on the local device; the anchor does not trust remote task metadata alone |
| Privileged broker unavailability blocks legitimate elevated tasks | Low | Fail-closed by design; non-elevated tasks continue; user is notified of degraded state |

---

## 7. Acceptance criteria coverage

- [x] **Runtime responsibilities defined** (Section 1)
- [x] **Docker role clarified (packaging vs. security boundary)** (Section 2)
- [x] **Resource access boundaries defined** (Section 3)
- [x] **Hive relationship documented** (Section 4)
- [x] **Risks listed** (Section 6.2)

## Related

^[source-materials/mirrors/doctrine/alice-hosting-anchor-runtime.md]
