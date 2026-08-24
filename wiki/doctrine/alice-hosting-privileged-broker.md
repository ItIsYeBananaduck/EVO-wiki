---
title: alice-hosting-privileged-broker
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/alice-hosting-privileged-broker.md
updated: 2026-07-24
---

# EVOconnect Privileged Execution Broker Design

> Output of [EVOC-325](https://linear.app/lsctech/issue/EVOC-325/02-design-privileged-execution-broker-for-connect): design for the narrow, non-agentic privileged execution broker that handles elevated actions in EVOconnect.

This document is referenced by [EVOC-326](https://linear.app/lsctech/issue/EVOC-326/03-evaluate-alice-as-user-profile-isolation-model) (OS isolation evaluation) and [EVOC-323](https://linear.app/lsctech/issue/EVOC-323/04-map-governance-and-audit-rules-across-hosting-models) (governance mapping). It does not implement the broker; it defines the design constraints, flows, and interfaces.

---

## 1. Broker responsibilities

The privileged execution broker is a separate, tightly-scoped system component. It is **not** part of Alice's runtime. Alice requests elevated actions from the broker; the broker independently evaluates and executes or rejects them.

**Core responsibilities:**

1. **Gate elevated actions** — accept or reject action requests from Alice based on the Delegator's authorization state and a fixed set of allowed action classes.
2. **Validate authorization** — independently verify that the originating task holds a valid, scoped `ToolGrant` in `AuthorizedByMethod` or `AuthorizedByTalent` state before executing any action.
3. **Execute within class boundaries** — perform only actions belonging to a declared, enumerated action class. Reject anything outside the class list.
4. **Log every interaction** — record every request, approval, denial, and execution result in an append-only audit log separate from Alice's logs.
5. **Enforce temporal and scope bounds** — respect TTL, max-step limits, and task-scoped boundaries on every grant. Never hold standing credentials.

**What the broker is not:**

- Not a shell or command interpreter. It does not accept arbitrary commands.
- Not agentic. It does not plan, reason, or decide what is "safe." It applies fixed rules.
- Not an extension of Alice's tool surface. Alice's tools and the broker's action classes are disjoint.

---

## 2. Elevation flow (end-to-end)

The flow below describes a single elevated action request from initiation to completion.

```
Alice Runtime                    Broker                         OS / System
    |                               |                               |
    |  1. Request(action_class,     |                               |
    |     params, task_id,          |                               |
    |     tool_grant_ref)           |                               |
    |------------------------------>|                               |
    |                               |  2. Validate:                 |
    |                               |     a. task_id exists          |
    |                               |     b. task in Authorized*     |
    |                               |        state                   |
    |                               |     c. ToolGrant is valid,     |
    |                               |        not expired, covers     |
    |                               |        this action class       |
    |                               |     d. action_class is in      |
    |                               |        allowed list            |
    |                               |     e. params match class      |
    |                               |        schema                  |
    |                               |                               |
    |                               |  3a. REJECT (if any check      |
    |  <---[Denial + reason]--------|      fails)                    |
    |                               |                               |
    |                               |  3b. APPROVE                   |
    |                               |     Check approval gate:       |
    |                               |     - Talent-bound context:    |
    |                               |       auto-approve             |
    |                               |     - Method-bound context:    |
    |                               |       require user             |
    |                               |       confirmation             |
    |                               |                               |
    |                               |  4. Execute action with        |
    |                               |     minimum-privilege           |
    |                               |     credentials                |
    |                               |----------------------------->  |
    |                               |                               |
    |                               |  5. Receive result             |
    |                               |  <-----------------------------  |
    |                               |                               |
    |                               |  6. Write audit log entry      |
    |                               |     (before returning result)  |
    |                               |                               |
    |  <---[Result or error]--------|  7. Return result to Alice     |
    |                               |                               |
```

### Flow rules

- **Steps 2a-2e are all mandatory.** A missing or invalid field causes immediate rejection.
- **Step 3b approval gate** follows the Delegator's two authorization paths:
  - `AuthorizedByTalent`: The Talent snapshot includes the action class in its declared scope. No per-action user prompt is required.
  - `AuthorizedByMethod`: The user must explicitly confirm the elevated action before the broker executes it. The broker presents the action class, parameters, and target to the user for approval.
- **Step 6 happens before step 7.** The audit log entry must be durable before the result is returned. If logging fails, the action result is NOT returned to Alice and the action is treated as failed. The broker enters a degraded state and refuses further actions until logging is restored.
- **No chaining.** Each request is independent. The broker does not batch, pipeline, or auto-chain actions. Alice must submit separate requests for separate actions.

---

## 3. Allowed action classes

The broker operates on a **fixed, enumerated** list of action classes. Each class defines a narrow category of elevated operation, a parameter schema, and a privilege ceiling.

| Action class | Description | Example operations | Privilege ceiling |
|---|---|---|---|
| `fs.protected-read` | Read files outside Alice's sandbox | Read a config file in a system directory | Read-only access to the specific path |
| `fs.protected-write` | Write files outside Alice's sandbox | Update a config file the user has approved | Write access to the specific path; no directory creation outside declared scope |
| `process.manage` | Start, stop, or restart a user-space process | Restart a background service after config change | User-space process control only; no system services |
| `network.listen` | Bind a listening port | Start a local dev server on a specified port | Bind to localhost only; port must be in allowed range |
| `credential.read` | Read a credential from the system keychain or secrets store | Retrieve an API key for a task-authorized integration | Read-only; credential is passed to Alice's tool, not displayed or logged |
| `system.query` | Query system state (disk, memory, installed packages) | Check available disk space before a large operation | Read-only; no mutations |
| `os.notify` | Send an OS-level notification to the user | Alert the user that a long-running task completed | Notification only; no actions attached |

### Class constraints

- **Closed set.** Any request with an `action_class` not in this list is rejected. There is no `shell.exec`, `command.run`, or equivalent.
- **Schema-enforced parameters.** Each class has a fixed parameter schema. The broker validates parameters against the schema before execution. Unknown parameters are rejected.
- **No parameter interpretation.** The broker does not parse, expand, or interpret parameter values beyond schema validation. It does not evaluate expressions, globs, or templates.
- **Additive only.** New action classes may be added in future versions. Existing classes may be narrowed but not widened without a new version.

---

## 4. Delegator integration points

The broker integrates with the existing Delegator system at the following points:

### 4.1 ToolGrant validation

Before executing any action, the broker queries the Delegator to confirm:

| Check | Source | Failure mode |
|---|---|---|
| Task exists and is active | Task Manager | Reject: task not found or already completed |
| Task is in `AuthorizedByMethod` or `AuthorizedByTalent` state | Delegator state machine | Reject: task not in executable state |
| ToolGrant is valid and not expired | Delegator ToolGrant store | Reject: grant expired or revoked |
| ToolGrant scope covers the requested action class | ToolGrant `allowed_tools` mapped to broker action classes | Reject: action class not in grant scope |
| ToolGrant TTL and step count are within limits | ToolGrant metadata | Reject: limits exceeded |

The broker does **not** modify the Delegator's state. It is a read-only consumer of authorization state.

### 4.2 Method and Talent binding

- For `AuthorizedByMethod` tasks: the broker checks that the task's method declares the requested action class in its `required_tools` (mapped to broker classes). The user must confirm each action.
- For `AuthorizedByTalent` tasks: the broker checks that the Talent's immutable manifest includes the action class. No per-action user confirmation is required because the Talent was pre-approved.

The broker never accepts a request that lacks a `method_id` or `talent_id` reference. Ad-hoc elevated actions are not permitted.

### 4.3 Task lifecycle integration

The broker's actions are part of the enclosing task's lifecycle:

- The broker executes only during the `Executed` phase of the task lifecycle.
- Broker audit log entries contribute to the task's `Logged` state. A task cannot transition to `Completed` until all broker log entries for that task are durable.
- If the user revokes approval (transitions the task out of an authorized state), the broker immediately rejects any pending or new requests for that task.

### 4.4 Swarm coordination

When the anchor runtime coordinates Swarm nodes:

- Swarm nodes **cannot** invoke the broker. Only the local Alice runtime (the anchor) may submit broker requests.
- If a Swarm node needs an elevated action, it returns an advisory result to the anchor. The anchor then decides whether to submit a broker request through normal authorization.

---

## 5. Logging and auditability requirements

### 5.1 Log structure

Every broker interaction produces a structured log entry with the following minimum fields:

| Field | Type | Description |
|---|---|---|
| `broker_event_id` | UUID | Unique identifier for this broker event |
| `timestamp` | ISO 8601 | When the event occurred |
| `event_type` | enum | `request`, `validation_pass`, `validation_fail`, `user_prompt`, `user_approve`, `user_deny`, `execute`, `execute_success`, `execute_failure`, `logging_degraded` |
| `task_id` | string | The originating task's ID |
| `action_class` | string | The requested action class |
| `params_hash` | SHA-256 | Hash of the request parameters (not the raw values, to avoid logging secrets) |
| `authorization_path` | enum | `method` or `talent` |
| `method_or_talent_ref` | string | The method_id or talent_id that authorized the action |
| `tool_grant_id` | string | The ToolGrant ID used for validation |
| `outcome` | enum | `approved`, `denied`, `executed`, `failed` |
| `denial_reason` | string \| null | Reason for denial if the request was rejected |
| `execution_duration_ms` | integer \| null | How long the action took to execute |

### 5.2 Log properties

- **Append-only.** Broker logs are written to an append-only store. Entries are never modified or deleted during normal operation.
- **Separate from Alice's logs.** The broker maintains its own log file/store, distinct from Alice's task logs and the Task Manager's audit log. This prevents Alice from modifying broker audit records.
- **Durable before response.** The log entry for an action must be fsynced/durable before the broker returns the result to Alice. If durability cannot be confirmed, the broker enters degraded mode.
- **User-inspectable.** The primary user must be able to view broker logs through the Task Manager UI. Logs are never hidden or restricted to a separate profile.
- **Credential redaction.** For `credential.read` actions, the log records the credential name/identifier but never the credential value. `params_hash` covers the full parameters; raw parameters are not stored for credential actions.

### 5.3 Log retention

Broker logs follow the same retention policy as the Task Manager's audit logs. They must be available for at least the retention period defined by the deployment's audit policy.

---

## 6. Constraints preventing unrestricted admin execution

These constraints are **absolute and non-negotiable**. Any deployment that violates these constraints is non-compliant.

### 6.1 No admin identity

- The broker process runs as a **non-admin, non-root** user. It uses per-action, scoped privilege escalation (e.g., `polkit` actions, macOS authorization services, or scoped `sudo` rules) rather than running as a privileged user.
- The broker does not store or cache admin credentials. Each escalation is individually authorized by the OS-level privilege system.

### 6.2 No open-ended commands

- The broker does not expose a shell, command-line interface, or arbitrary code execution endpoint.
- There is no `execute(command: string)` action class. All action classes have typed, schema-validated parameters.
- The broker does not interpret natural language, evaluate expressions, or construct commands from templates.

### 6.3 No standing credentials

- The broker does not hold persistent API keys, tokens, or credentials that grant broad access.
- Credentials required for specific action classes (e.g., `credential.read` accessing a keychain) are obtained per-action through the OS credential system and released immediately after use.
- ToolGrants have TTL and step-count limits. When a grant expires, the broker rejects all further requests under that grant.

### 6.4 No action chaining

- The broker does not support "if action A succeeds, automatically do action B."
- Each request is independent. Alice must submit separate requests for separate actions.
- The broker does not maintain conversational state between requests. Each request is evaluated from scratch against the current Delegator state.

### 6.5 No scope expansion

- The broker cannot add new action classes at runtime. The action class list is fixed at deployment and changes only through a versioned release.
- The broker cannot widen an existing action class's parameter schema at runtime.
- A ToolGrant's scope cannot be expanded after creation. Narrowing is permitted (by revocation); widening requires a new grant.

### 6.6 Fail-closed behavior

- If the broker cannot reach the Delegator to validate a request, it **rejects** the request. It does not cache authorizations or operate in an offline mode.
- If the broker's logging system is unavailable, it enters degraded mode: it returns results for in-flight actions but rejects all new requests until logging is restored.
- If the broker process crashes, no elevated actions can execute until it restarts. Alice does not gain elevated capabilities when the broker is down.

### 6.7 User override

- The user can stop the broker at any time. Stopping the broker immediately prevents all elevated actions.
- The user can revoke any ToolGrant, which the broker respects on the next validation check.
- The user can inspect the full broker log at any time through the Task Manager.

---

## Acceptance criteria coverage

- [x] **Broker responsibilities defined** (Section 1)
- [x] **Elevation flow documented end-to-end** (Section 2)
- [x] **Allowed action classes listed** (Section 3)
- [x] **Logging requirements defined** (Section 5)
- [x] **Explicit constraints preventing unrestricted admin execution documented** (Section 6)

## Related

^[source-materials/mirrors/doctrine/alice-hosting-privileged-broker.md]
