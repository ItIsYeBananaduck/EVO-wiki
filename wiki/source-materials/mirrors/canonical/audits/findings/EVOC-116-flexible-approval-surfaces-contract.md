---
type: audit-finding
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-116 — Flexible Approval Surfaces Contract

> Status: Draft contract (architecture only; no UI implementation).
> Date: 2026-04-04.
> Parent: EVOC-194 — Connect Supervision and Learning Surface.
> Informed by: EVOC-71, EVOC-49, EVOS1-35.

## 1) Purpose

Define a single approval contract that can be completed from multiple user surfaces without breaking governance semantics or blocking user flow.

This contract standardizes approval behavior across:

- chat approval prompts,
- task manager approvals,
- notification-based approvals,
- inline approvals in terminal/browser execution views,
- and optional "approve at task start" behavior for Methods.

## 2) Core invariants

1. **One approval truth, many surfaces**: every surface must resolve the same canonical approval record.
2. **No implicit approval**: approval requires explicit user action and identity attribution.
3. **First decision wins**: once an approval request is resolved (approved, denied, expired, or superseded), all surfaces must converge to that terminal state.
4. **Cross-surface consistency**: labels, status, and decision semantics must not conflict between surfaces.
5. **Fail-closed**: if approval cannot be captured/verified, execution remains blocked.
6. **Delegator authority preserved**: this issue adds surfaces only; it does not alter Delegator policy or rule precedence.
7. **Mobile and desktop parity**: all required approval outcomes must be achievable on both form factors.

### 2.1 Risk tier and policy alignment

The `riskTier` field must use one of these valid values:

- `low`: actions with minimal impact or risk.
- `medium`: actions requiring user review but with bounded risk.
- `high`: actions with potentially significant impact requiring explicit approval.

**Mapping to EVOS1-35 safety classification:**

| `riskTier` value | EVOS1-35 classification | Default approval behavior |
|------------------|-------------------------|---------------------------|
| `low` | Auto-approved | No explicit approval required (may log/audit only) |
| `medium` | Approval-required | Explicit approval required before execution |
| `high` | Approval-required or Forbidden | Explicit approval required; some high-risk actions may be forbidden by policy. See precedence order below for decision rules. |

**Decision rules for `high` riskTier:**

For actions with `riskTier: high`, the final approval decision is determined by the following precedence order:

1. **EVOS1-35 SafetyRules classification** is evaluated first:
   - If SafetyRules returns `"forbidden"`, the action is marked as **Forbidden** (no approval surface may override; fail-closed per invariant #5).
   - If SafetyRules returns `"approval-required"` or `"auto-approved"`, proceed to step 2.

2. **Method-level configuration overrides** (if present):
   - If the Method or task has explicit configuration overriding the risk tier (e.g., method-level allow-list or deny-list), apply that override.
   - If an override results in `"forbidden"`, the action is **Forbidden**.
   - If an override results in `"approval-required"`, proceed to step 3.
   - If no override is present, proceed to step 3.

3. **Policy/risk-score threshold evaluation**:
   - Apply the policy engine or organization-level policy to evaluate configured risk score thresholds for `high` risk tier.
   - If the policy determines the action exceeds the threshold for `high` risk, mark as **Forbidden**.
   - Otherwise, mark as **Approval-required**.

**Interaction with `riskTier` (task/method-level) and per-action SafetyRules:**

- The `riskTier` field is a task- or method-level classification indicating the general risk category of the action.
- Per-action SafetyRules (EVOS1-35) evaluate each individual action dynamically and may return a stricter classification than the task/method-level `riskTier`.
- When a conflict occurs (e.g., task has `riskTier: medium` but a specific action is classified as `"forbidden"` by SafetyRules), the stricter SafetyRules outcome takes precedence.
- This ensures that even if a task is generally low or medium risk, individual high-risk or forbidden actions within that task are still subject to EVOS1-35 governance.

**Interaction with `requiresApprovalReason`:**

- `requiresApprovalReason` is **required** when `riskTier` is `medium` or `high`.
- `requiresApprovalReason` must be a non-empty string documenting the policy rule or risk condition that triggered the approval requirement (e.g., "EVOS1-35: network access requires approval", "user-configured guardrail: API keys in scope").
- For `low` risk tier, `requiresApprovalReason` may be null or omitted unless a user-configured rule mandates approval.

**Precedence and conflict resolution with EVOS1-35:**

1. EVOS1-35 safety rules take precedence over Method-level `approveAtTaskStart` preferences.
2. If EVOS1-35 classifies an action as "forbidden", no approval surface may override it (fail-closed).
3. If EVOS1-35 requires per-action approval, `approveAtTaskStart` semantics must defer to per-action gating.
4. This contract implements approval surfaces only; it inherits and enforces EVOS1-35 policy decisions without altering rule evaluation logic or precedence order.
5. In case of conflict (e.g., Method requests `approveAtTaskStart` for a high-risk action that EVOS1-35 mandates per-action approval), the stricter EVOS1-35 rule must govern.

## 3) Approval lifecycle model

### 3.1 Canonical states

- `pending`
- `approved`
- `denied`
- `expired`
- `superseded`

### 3.2 State semantics

1. **`pending`**
   - Approval request exists and awaits decision.
   - Execution path is blocked where approval is required.
2. **`approved`**
   - Authorized actor explicitly approved.
   - The blocked path may continue according to existing execution rules.
3. **`denied`**
   - Authorized actor explicitly denied.
   - Blocked path transitions to denial handling/fallback.
4. **`expired`**
   - Request timed out before decision.
   - Must behave as deny-by-default for gated execution.
   - **System actions when an approval expires:**
     1. The associated request/task must transition to a terminal denied/failed execution state.
     2. A standardized denial event (`approval.expired`) must be emitted with `reason="expired"` for audit/telemetry.
     3. All decision transitions are blocked (similar to `superseded`); the approval record becomes immutable.
   - These semantics enforce invariant #5 (fail-closed) and ensure the execution engine can deterministically halt gated paths.
5. **`superseded`**
   - Request replaced by newer request/version.
   - Must not accept further decisions.

### 3.3 Allowed transitions

- `pending -> approved`
- `pending -> denied`
- `pending -> expired`
- `pending -> superseded`

Disallowed:

- non-`pending` -> any other state,
- any transition that reopens a terminal request,
- direct `approved -> denied` or `denied -> approved` on the same request id.

## 4) Multi-surface completion contract

### 4.1 Supported approval surfaces

Each `pending` request must be renderable (or deeplink-resolvable) from:

1. **Chat surface**: conversational card/prompt with decision controls.
2. **Task manager surface**: task-level approval list/detail with decision controls.
3. **Notification surface**: actionable notification or deep link into actionable surface.
4. **Inline execution surfaces**: terminal/browser inline controls bound to active execution context.

### 4.2 Surface behavior requirements

All surfaces must:

1. display shared request identity (`approvalId`) and current status,
2. expose canonical decision actions (`approve`, `deny`) when `pending`,
3. disable or hide decision actions for non-`pending` states,
4. confirm decision outcome with timestamp and actor,
5. refresh when a decision is made from any other surface.

### 4.3 Conflict handling

If two surfaces attempt decisioning concurrently:

1. backend resolution must be atomic,
2. first accepted decision sets terminal state,
3. losing attempts receive deterministic stale-state response,
4. all surfaces must reconcile to terminal state without requiring manual refresh loops.

**Atomic conflict-resolution mechanism:**

- The backend must use **optimistic locking** via a version or timestamp field on the approval decision record (e.g., `decision_version` integer that increments on each state change, or `updated_at` timestamp for comparison).
- Clients/surfaces must supply the current version when submitting a decision (e.g., `expectedVersion` or `ifVersionMatches`).
- `DecisionService.updateDecision` (or equivalent backend method) must perform a **conditional write** that:
  1. Checks the supplied version against the current database record version.
  2. If the version matches, applies the state transition and increments the version atomically.
  3. If the version mismatches (indicating a concurrent update), rejects the write and returns a deterministic stale-state response (e.g., HTTP 409 Conflict with `{error: "decision_version_mismatch", currentState: "approved", currentVersion: N}`).
- For rare high-contention paths, the backend may use a **serializable database transaction** or **row-level lock** as a fallback to guarantee atomicity.
- The stale-state response must include:
  - the current terminal state (`approved`, `denied`, `expired`, or `superseded`),
  - the current version number,
  - metadata allowing the client to reconcile (e.g., `decidedByActorId`, `decidedAt`, `decisionSurface`).
- All surfaces must handle the stale-state response by fetching the current terminal state (or using the returned metadata) without entering manual refresh loops or retry logic that could cause further contention.

## 5) Optional Method behavior: approve at task start

`approveAtTaskStart` is an optional Method-level mode that allows one explicit approval decision to unlock task start when policy allows.

Rules:

1. Option must be explicit and visible before decision.
2. Decision scope must be clearly bounded (task id + method version).
3. Mode must not bypass risk-tier approvals that require per-action approvals by policy.
4. Audit trail must record that approval used `approveAtTaskStart` semantics.
5. If scope changes materially (task content/risk change), existing grant must be invalidated and a new request issued.

**Explicit invalidation triggers for `approveAtTaskStart` scope:**

An existing `approveAtTaskStart` grant is invalidated (and a new approval request must be issued) when any of the following triggers occur:

1. **Task description or instructions are edited** (any change to task prompt, goals, or user-provided instructions).
2. **Task parameters or input data change** (e.g., new arguments, updated context, modified input variables).
3. **Associated Method version updates** (task rebinds to a different `methodVersion` or the Method definition itself changes).
4. **Risk tier increases** or policy-required per-action approvals are added (e.g., EVOS1-35 rule changes from `low` to `medium`/`high`, or a new per-action approval requirement is applied).
5. **Target resource identifiers change** (e.g., different file paths, API endpoints, database tables, or external service targets are added/modified).

**Recording and enforcement:**

- Invalidation events must be recorded in the audit trail with:
  - `approvalId` of the invalidated grant,
  - which trigger(s) fired (e.g., `trigger: "task_description_edited"`),
  - timestamp and actor (if user-initiated change) or system event source.
- When an invalidation trigger fires, the system must:
  1. Mark the existing approval as `superseded`.
  2. Create a new `pending` approval request reflecting the updated scope (new task id + method version or updated parameters).
  3. Block execution until the new approval is obtained.
- The `approveAtTaskStart` mode and decision scope (task id + method version) bind these invalidation rules; they do not apply to `per_request` mode, which re-requests approval on every action regardless of scope changes.

## 6) Required approval payload

Every approval request must include:

- `approvalId`
- `executionId`
- `taskId` (nullable only if not task-bound)
- `methodId` (nullable)
- `methodVersion` (nullable)
- `requiresApprovalReason`
- `riskTier`
- `requestedActionsSummary`
- `state` (`pending` | `approved` | `denied` | `expired` | `superseded`)
- `requestedAt`
- `expiresAt` (nullable when no expiry policy)
- `decidedAt` (nullable while pending)
- `decidedByActorId` (nullable while pending)
- `decisionSurface` (nullable while pending)
- `decisionMode` (`per_request` | `approve_at_task_start`, nullable while pending)
- `surfaceHints` (surfaces that can render decision immediately)
- `decision_version` (integer, increments on each state change; used for optimistic locking)

Validation rules:

1. Decision fields (`decidedAt`, `decidedByActorId`, `decisionSurface`, `decisionMode`) must be all-null while `pending`.
2. Decision fields must be all-present for `approved` and `denied`.
3. For `expired` state (system-driven timeout):
   - `decidedAt` must be present (timestamp of expiry transition).
   - `decisionMode` must be present (inherited from the request; typically `per_request` or `approve_at_task_start`).
   - `decidedByActorId` must be null (no human actor for system-driven expiry).
   - `decisionSurface` must be null (system transition, not surface-initiated).
   - Rationale: Expiry is a system-driven terminal transition attributed to timeout policy, not actor decision.
4. For `superseded` state (request replaced by newer version):
   - `decidedAt` must be present (timestamp of superseding transition).
   - `decisionMode` must be present (inherited from the original request).
   - `decidedByActorId` may be null (if system-driven, e.g., scope invalidation) or present (if actor-initiated, e.g., user edits task triggering re-approval).
   - `decisionSurface` may be null (if system-driven) or present (if actor-initiated from a specific surface).
   - Rationale: Superseding may be triggered by system events (scope changes per section 5 invalidation triggers) or by actor edits; populate actor/surface fields only when superseding is actor-initiated.
5. `decisionMode = approve_at_task_start` is valid only when Method is marked eligible (see section 6.2).
6. `expiresAt < now` with no decision must transition to `expired` (where "now" is defined as the server's authoritative current time per section 6.3).

### 6.1 Schema integration plan

**Preferred approach: dedicated `ApprovalRecord` entity**

Approvals are lifecycle-shared (may be queried independently, referenced by multiple surfaces, and audited separately from task execution) and should be stored as a first-class entity.

**Schema definition for `ApprovalRecord`:**

- Primary key: `approvalId` (UUID or equivalent unique identifier).
- All 17 fields listed above (including `decision_version`).
- Foreign key: `methodExecutionRecordId` → `TaskExecutionRecord.methodExecutionRecordId` (establishes the relationship between approval and execution).
- Nullable foreign keys: `taskId`, `methodId`, `methodVersion` (nullable when approval is not task-bound or method-bound).
- State transition rules:
  - `state` starts as `pending`.
  - Valid transitions: `pending → {approved, denied, expired, superseded}`.
  - Terminal states (`approved`, `denied`, `expired`, `superseded`) are immutable; no further transitions allowed.
- Nullability and constraints:
  - `decidedAt`, `decidedByActorId`, `decisionSurface`, `decisionMode` are nullable while `state = pending`; required (non-null) for terminal decision states (`approved`, `denied`).
  - `expiresAt` nullable when no expiry policy; server checks `expiresAt < now` to auto-transition to `expired`.
  - `decision_version` defaults to 0 on creation; increments atomically on every state change (used for optimistic locking per section 4.3).

**Alternative approach (not recommended): embed approvals in `TaskExecutionRecord`**

If approvals are tightly coupled to execution and never queried/shared independently, add an optional `approvals` JSON array or nested table field to `TaskExecutionRecord`. This approach is less flexible for cross-surface querying and independent audit trails.

**Impact on `TaskMethodBinding`:**

- `TaskMethodBinding` should include method-level eligibility metadata (e.g., `approvalModes: ["per_request", "approve_at_task_start"]` or a boolean `supportsApproveAtTaskStart`).
- This metadata is used during validation (rule 3 above) to verify that `decisionMode = approve_at_task_start` is valid for the Method.
- No structural changes to `TaskMethodBinding` are required if eligibility is stored as metadata on the Method definition itself (preferred).

**Phase 8 migrations and service boundaries:**

- Phase 8 (or equivalent backend migration phase) must:
  1. Create the `ApprovalRecord` table/entity with all 17 fields.
  2. Add foreign key relationship `ApprovalRecord.executionId → TaskExecutionRecord.executionId`.
  3. Add indexes on `state`, `executionId`, `taskId`, and `expiresAt` for efficient querying.
  4. Implement state transition logic in `DecisionService` or equivalent service layer.
  5. Ensure `decision_version` optimistic locking is enforced in write operations (per section 4.3).
- Service boundaries:
  - `ApprovalService` (or `DecisionService`) owns CRUD and state transitions for `ApprovalRecord`.
  - `ExecutionService` reads approval state to gate execution but does not mutate approval records.
  - Event emitters for `approval.*` events must be triggered by `ApprovalService` on state changes.

### 6.2 Method eligibility schema

To support validation rule 3 (ensuring `decisionMode = approve_at_task_start` is used only with eligible Methods), every Method definition must include eligibility metadata.

**Schema addition to Method definition:**

- Field: `approvalModes` (array of strings, e.g., `["per_request"]` or `["per_request", "approve_at_task_start"]`).
- Location: Method metadata (stored alongside `methodId`, `methodVersion`, and other Method properties).
- Allowed values:
  - `"per_request"`: Method supports standard per-action approval (always available).
  - `"approve_at_task_start"`: Method supports upfront approval that unlocks task start (optional, requires eligibility).
- Default: `["per_request"]` (if not specified, Method is not eligible for `approveAtTaskStart`).

**Eligibility rules for `approve_at_task_start`:**

A Method is eligible for `approveAtTaskStart` mode only if it meets all of the following criteria:

1. **Deterministic/idempotent semantics**: Method actions must be predictable and repeatable (no hidden side effects or non-deterministic behavior that would invalidate upfront approval).
2. **Max risk tier or Delegator threshold**: Method must not exceed a maximum risk tier (e.g., no `high` risk Methods unless explicitly allow-listed) or must satisfy Delegator-configured thresholds (e.g., EVOS1-35 rules permit task-level approval for this Method).
3. **Bounded scope**: Method scope (inputs, outputs, resource access) must be fully known at task start; no dynamic scope expansion that would invalidate the approval grant.
4. **No per-action policy override**: EVOS1-35 or other governance policies must not mandate per-action approval for this Method's risk tier or action types.

**Validation logic:**

When validating an approval request with `decisionMode = approve_at_task_start`:

1. Retrieve the Method definition using `methodId` and `methodVersion`.
2. Check that `"approve_at_task_start"` is present in `method.approvalModes`.
3. If not present, reject the request with a validation error (e.g., `{error: "method_not_eligible_for_approve_at_task_start", methodId, methodVersion}`).
4. If present, proceed with approval request creation and validation.

**Example Method metadata:**

```json
{
  "methodId": "file_search_v2",
  "methodVersion": "2.1.0",
  "approvalModes": ["per_request", "approve_at_task_start"],
  "eligibilityRationale": "Deterministic file search with bounded scope; no dynamic resource expansion."
}
```

### 6.3 Server time definition for expiry checks

To provide fail-closed semantics and handle clock skew, all expiry comparisons must use the server's authoritative time:

- **"now" is defined as**: the server-side UTC clock time at the moment of the expiry check (using `Date.now()`, `clock_gettime(CLOCK_REALTIME)`, or equivalent monotonic/UTC time source).
- **Expiry checks**:
  - When `expiresAt` is non-null, the server must evaluate `expiresAt < now` (where `now` is server time).
  - If true and `state = pending`, the approval must automatically transition to `expired`.
  - Client-provided timestamps (e.g., from mobile devices with clock skew) must **not** be used for expiry decisions.
- **Monotonic guarantees**:
  - For critical transition logic (e.g., checking expiry during a decision attempt), the server should use a monotonic clock source to avoid time drift or NTP adjustments causing inconsistent expiry behavior.
  - If the server clock is adjusted backward, existing `expiresAt` values must remain valid (no retroactive expiry extensions).
- **Transition enforcement**:
  - Expiry transitions (`pending → expired`) must be checked:
    1. On every decision attempt (before accepting `approve` or `deny` actions).
    2. Periodically via a background job or trigger (e.g., every 60 seconds, or on next database read for pending approvals).
    3. When querying approval status for execution gating.
  - Once transitioned to `expired`, the approval record is immutable and must emit `approval.expired` event with `reason="expired"`.

## 7) Event contract

Required events:

- `approval.requested`
- `approval.presented` (surface-specific)
- `approval.approved`
- `approval.denied`
- `approval.expired`
- `approval.superseded`
- `approval.stale_decision_rejected`

Minimum event fields:

- `approvalId`
- `executionId`
- `state`
- `surface`
- `timestamp`
- `actorId` (nullable for system events)
- `decisionMode` (nullable where not applicable)

Consistency rules:

1. terminal decision event must be emitted once per `approvalId`,
2. stale attempts must emit `approval.stale_decision_rejected`,
3. presentation events must never imply authority change.

## 8) UX and platform parity requirements

1. Mobile and desktop must both support:
   - seeing pending approvals,
   - taking approve/deny action,
   - viewing final decision details.
2. Notification-driven approvals may open into another surface if direct action cannot be completed safely on device/OS.
3. Terminal/browser inline approvals must preserve context so users understand exactly what they are approving.
4. Copy may vary by density, but decision meaning must remain equivalent.

## 9) Guardrails and non-goals

Prohibited:

- implicit approval from message reply, navigation, or inactivity,
- surface-specific approval semantics that diverge from canonical states,
- accepting decisions without actor identity and timestamp,
- using `approveAtTaskStart` to bypass higher-governance approval requirements,
- changing Delegator policy semantics in this issue.

Non-goals:

- redesign of Delegator rules,
- visual design system details,
- push transport/provider implementation details,
- advanced escalation policy redesign.

## 10) Acceptance criteria

This issue is complete when:

1. Approval can be completed from chat, task manager, notification flow, and terminal/browser inline surfaces.
2. Approval records resolve consistently regardless of where decision is made.
3. Mobile and desktop can both complete required approval outcomes.
4. Optional `approveAtTaskStart` behavior is contract-defined with explicit scope and safeguards.
5. Contract preserves existing Delegator governance semantics.
6. **Alignment with EVOC-49 (execution states and invariants)**:
   - Approval state transitions (`pending → approved/denied/expired/superseded`) must be validated against EVOC-49's execution state model.
   - Gated execution must respect approval states: `approved` allows continuation, `denied`/`expired`/`superseded` halt execution, per EVOC-49 invariants.
   - The `ApprovalRecord` schema and `ExecutionService` integration must ensure that execution cannot proceed without a valid `approved` state when approval is required (enforcing EVOC-49's execution gating semantics).
7. **Alignment with EVOS1-35 (Delegator governance and safety semantics)**:
   - All risk tier mappings (section 2.1) and `requiresApprovalReason` logic must enforce EVOS1-35's three-tier safety classification (auto-approved / approval-required / forbidden).
   - EVOS1-35 precedence rules must govern conflict resolution: if EVOS1-35 forbids an action, no approval surface may override; if EVOS1-35 requires per-action approval, `approveAtTaskStart` must defer to per-action gating.
   - Method eligibility for `approveAtTaskStart` (section 6.2) must respect EVOS1-35 Delegator thresholds and veto/override rules.
   - Invalidation triggers (section 5) must include EVOS1-35 policy changes (e.g., risk tier increases mandated by Delegator rule updates).
   - Acceptance testing must verify that implementing optional `approveAtTaskStart` and multi-surface approval does not weaken or bypass EVOS1-35 Delegator governance (including veto/override rules, per-action approval requirements, and forbidden action enforcement).