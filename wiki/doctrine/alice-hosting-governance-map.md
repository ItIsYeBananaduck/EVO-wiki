---
title: alice-hosting-governance-map
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/alice-hosting-governance-map.md
updated: 2026-07-24
---

# EVOconnect Always-On Alice Governance Compliance Map

> Output of [EVOC-323](https://linear.app/lsctech/issue/EVOC-323/04-map-governance-and-audit-rules-across-hosting-models): maps EVOconnect governance and audit rules across all candidate Alice hosting models.

This document is an input to the v1 strategy selection in [EVOC-327](https://linear.app/lsctech/issue/EVOC-327/05-select-v1-always-on-alice-deployment-strategy). It does not implement governance changes; it records compliance requirements and violations for each candidate model.

## Candidate hosting models

| Model | Description | Depends on |
|-------|-------------|------------|
| A. Standard local runtime | Alice runs as a normal app process when the user explicitly launches it. No persistent background execution. | None |
| B. Native desktop anchor | Alice runs as a user-space background service / daemon on the user's desktop, launched at login. | EVOC-324 |
| C. Docker self-hosted anchor | Alice runs inside a user-managed Docker container on the user's own machine. | EVOC-324 |
| D. Privileged supervised execution broker | A separate, tightly-scoped broker handles elevated actions that Alice cannot perform directly. | EVOC-325 |
| E. Alice-as-user-profile | Alice runs under a separate OS user account as a desktop automation isolation boundary. | EVOC-325, EVOC-326 |

## Governance principles (source)

All models must comply with the EVOconnect governance principles defined in the active doctrine:

- [MOC EVOconnect — Delegator](../../../smartdocs/doctrine/active/MOC%20EVOconnect%20%E2%80%94%20Delegator.md)
- [EVOconnect — Delegator Talent Verification Doctrine](../../../smartdocs/doctrine/active/EVOconnect%20%E2%80%94%20Delegator%20Talent%20Verification%20Doctrine.md)
- [Task Audit Log Minimum Fields](../../../smartdocs/doctrine/active/Task%20Audit%20Log%20Minimum%20Fields.md)
- [Task Chain Definition](../../../smartdocs/doctrine/active/Task%20Chain%20Definition.md)

Core requirements used in this map:

1. **Delegator enforcement**: Every task must be routed through the Delegator. Tools are denied unless the task is in `AuthorizedByMethod` or `AuthorizedByTalent` state with a valid, scoped `ToolGrant`.
2. **Full task lifecycle**: Tasks must follow the canonical lifecycle `Created → Reviewed → Approved → Executed → Logged → Completed`. No state may be skipped.
3. **Method binding**: Tasks must reference a `method_id` (or `talent_id`). Tool grants are scoped to `method.required_tools` and the immutable Talent snapshot. Ad-hoc tool access is not permitted.
4. **Auditability**: Each executed task must record Task ID, title, timestamps, authorization path, method/Talent reference, tool call summary, outcome, and user-visible artifacts.
5. **No implicit autonomy**: No auto-execution without approval or a valid Talent. No Talent enable/disable toggle; only revoke/recreate.
6. **Talents are immutable snapshots**: A promoted Talent cannot be edited in place; any change creates a new version.

## Compliance mapping per model

### A. Standard local runtime

| Principle | Assessment | Notes |
|-----------|------------|-------|
| Delegator enforcement | Compliant | Alice is a normal app process. The Delegator runs in-process or via a local coordinator. Task state machine is fully enforced. |
| Task lifecycle | Compliant | Full lifecycle is natural because the user is present and drives every transition. |
| Method binding | Compliant | Tasks are created explicitly with a method or Talent reference. |
| Auditability | Compliant | All task records are local to the user's device. Minimum audit fields are stored. |
| Unsafe patterns | — | User may leave the app mid-task; the Task Manager must persist state so execution can resume safely. |
| Compliance requirements | — | Maintain Delegator in the foreground loop; do not allow invisible background execution. |

**Violations identified**: None for the current user-present workflow. This model is non-compliant only if extended to run invisible tasks without approval or Talent.

---

### B. Native desktop anchor

| Principle | Assessment | Notes |
|-----------|------------|-------|
| Delegator enforcement | Compliant with conditions | The anchor process must still route every task through the Delegator. Background presence does not grant automatic tool access. |
| Task lifecycle | Compliant with conditions | Lifecycle is still enforced, but the `Reviewed`/`Approved` steps must not be skipped just because the user is not actively looking at the screen. |
| Method binding | Compliant | Background tasks must still reference an approved method or valid Talent. |
| Auditability | Compliant with conditions | Local audit logs must remain durable and tamper-evident. User must be able to inspect what happened while the anchor was running. |
| Unsafe patterns | — | Auto-execution of non-Talent tasks when the user is away. Silent retries that bypass approval. Running the anchor as root or with admin privileges. |
| Compliance requirements | 1. The anchor must hold a *scoped, time-limited* tool grant, not a permanent one. 2. Every background task must still appear in the Task Manager. 3. The user must be able to cancel any running task and revoke the grant immediately. 4. Audit logs must be written before the task is marked `Completed`. 5. No global admin rights. |

**Violations identified**:

- Running the anchor as a system service under an admin account would allow the model to bypass the Delegator's tool scoping.
- Executing any task that lacks a method or Talent reference violates Method binding.
- Skipping the `Approved` state for non-Talent tasks violates the lifecycle.
- Completing a task before writing its audit log violates the `Logged` state.

---

### C. Docker self-hosted anchor

| Principle | Assessment | Notes |
|-----------|------------|-------|
| Delegator enforcement | Compliant with conditions | Docker is a packaging and deployment convenience, not a security boundary. The Delegator must still enforce tool grants inside the container. |
| Task lifecycle | Compliant with conditions | Same lifecycle rules apply. Container restart must not erase pending approval states or in-progress task state. |
| Method binding | Compliant | The container must load the same method/Talent manifests and hashes as the host. |
| Auditability | Compliant with conditions | Audit logs must be written to a host-mounted volume or forwarded to the host so they survive container recreation. |
| Unsafe patterns | — | Treating the container as a privileged sandbox and running elevated commands inside it. Mounting the entire host filesystem. Storing secrets in container layers. Auto-executing tasks on container start without Delegator approval. |
| Compliance requirements | 1. Container must run as a non-root user. 2. Mount only the specific directories the workflow needs. 3. Do not grant `CAP_SYS_ADMIN` or Docker socket access. 4. Persist task state and audit logs outside the container. 5. Container image must include verified method/Talent manifests and validate their hashes. 6. The Delegator must be active inside the container; a container without Delegator enforcement is non-compliant. |

**Violations identified**:

- Running the container with `--privileged` or mounting `/var/run/docker.sock` gives the model unbounded host access and breaks Delegator enforcement.
- Any model-driven task that executes before the Delegator validates the task state violates the lifecycle.
- Storing method manifests or secrets in a mutable container filesystem without hash verification breaks Method binding.
- Completing a task before persisting the audit log to the host violates `Logged` state and auditability.

---

### D. Privileged supervised execution broker

| Principle | Assessment | Notes |
|-----------|------------|-------|
| Delegator enforcement | Compliant by design | The broker is the *only* path for elevated actions. Alice cannot execute elevated tools directly; she must request them through the broker, and the broker independently validates the request against the Delegator. |
| Task lifecycle | Compliant | The broker must not act until the task is in `AuthorizedByMethod` or `AuthorizedByTalent` state and the broker's own approval gate is satisfied. |
| Method binding | Compliant | The broker must accept only requests that name a method/Talent and an allowed action class. It must reject open-ended or undeclared actions. |
| Auditability | Compliant | Every broker request, approval, denial, and execution must be logged with the same minimum fields as a task, plus the broker's action class and reason. |
| Unsafe patterns | — | Allowing the broker to interpret natural language commands. Allowing the broker to chain actions without per-action approval. Giving the broker persistent credentials. Letting the broker decide what is "safe". |
| Compliance requirements | 1. Broker must check the Delegator/ToolGrant before executing. 2. Broker must have a fixed, enumerated list of allowed action classes. 3. Each action class must require a user approval or a Talent-bound context. 4. Broker must reject any request outside the allowed classes. 5. Broker logs must be append-only and separate from Alice's logs. 6. The broker must not hold standing credentials for open-ended use. 7. Broker must not expose a generic shell or command endpoint. |

**Violations identified**:

- A broker that executes any elevated command Alice asks for is not a broker; it is an unconstrained admin shell.
- A broker that skips the Delegator's `ToolGrant` check violates Delegator enforcement.
- A broker with persistent admin credentials that can be reused without per-action approval violates auditability and the lifecycle.

---

### E. Alice-as-user-profile isolation

| Principle | Assessment | Notes |
|-----------|------------|-------|
| Delegator enforcement | Compliant with conditions | The OS user profile isolates Alice's files and processes from the primary user. The Delegator must still run inside the Alice profile and enforce the same state machine. |
| Task lifecycle | Compliant with conditions | Cross-user actions (e.g., accessing the primary user's files) require explicit, auditable broker requests and approvals. |
| Method binding | Compliant | The Alice profile must load the same method/Talent manifests as the primary user. Method binding is not weakened by isolation. |
| Auditability | Compliant with conditions | Audit logs must be readable by the primary user. Cross-profile access must be logged on both sides. |
| Unsafe patterns | — | Using the profile boundary as a reason to bypass the privileged broker. Running the Alice profile with admin rights. Hiding tasks from the primary user's task history. Auto-executing cross-profile actions because "the profile is separate". |
| Compliance requirements | 1. The Alice profile must be a non-admin user. 2. Cross-profile file or UI automation must go through the privileged broker. 3. The primary user must see every task executed in the Alice profile. 4. Audit logs must be copied or forwarded to the primary user's storage. 5. Revoking the Alice profile must stop all background execution. 6. The profile must not be used to create a second, unaudited Delegator instance. |

**Violations identified**:

- Running the Alice profile as an administrator nullifies the isolation benefit and breaks Delegator enforcement.
- Any cross-profile action that does not route through the privileged broker violates the broker design and lifecycle.
- Keeping task history only inside the Alice profile, where the primary user cannot inspect it, violates auditability.

## Unsafe patterns summary

| Pattern | Why it violates governance | Affected models |
|---------|------------------------------|-----------------|
| Auto-execute non-Talent tasks while user is away | Skips `Approved` state | B, C, E |
| Run Alice or anchor as root/admin | Bypasses Delegator tool scoping | B, C, E |
| Privileged Docker container or Docker socket mount | Gives model unbounded host access | C |
| Broker with open-ended command interpretation | Breaks Method binding and Delegator enforcement | D |
| Persistent broker credentials without per-action approval | Breaks lifecycle and auditability | D |
| Cross-profile action without broker approval | Bypasses privileged broker | E |
| Mutable method manifests without hash verification | Breaks Method binding | All |
| Marking task `Completed` before audit log is written | Skips `Logged` state | All |
| Invisible task history not inspectable by user | Breaks auditability | B, C, E |
| Talent enable/disable toggle | Violates immutable Talent / revocation rule | All |

## Compliance requirements summary per model

| Model | Key requirements to remain compliant |
|-------|--------------------------------------|
| A. Standard local runtime | Keep Delegator in the execution loop; do not add invisible background execution. |
| B. Native desktop anchor | Scoped, time-limited tool grants; durable audit logs; non-admin user; cancellable tasks; no auto-execution of non-Talent tasks. |
| C. Docker self-hosted anchor | Non-root container; limited mounts; no privileged mode; state and audit logs on host; verified method manifests; Delegator active inside container. |
| D. Privileged supervised execution broker | Enumerated action classes; Delegator/ToolGrant check; per-action approval; no open-ended commands; separate append-only logs. |
| E. Alice-as-user-profile | Non-admin profile; broker-mediated cross-profile access; visible task history to primary user; forwarded audit logs; profile revocation stops execution. |

## Recommendations for EVOC-327

- **Do not select** any model that requires Alice or her runtime to run as root/admin.
- **Do not select** any model that permits auto-execution of non-Talent tasks without user approval.
- **Do not select** any model that hides task history from the user.
- **Require** the privileged broker for any model that involves cross-profile or elevated actions (models B, C, E).
- **Require** the same Delegator state machine and audit fields regardless of whether the runtime is in-process, a background service, a container, or a separate OS profile.

## Acceptance criteria coverage

- [x] Governance requirements mapped per hosting model (Sections A–E).
- [x] Violations identified for any non-compliant model (Violations listed under each model).
- [x] Unsafe patterns flagged (Unsafe patterns summary table).
- [x] Compliance requirements documented for each model (Compliance requirements summary table and per-model requirements).

## Related

^[source-materials/mirrors/doctrine/alice-hosting-governance-map.md]
