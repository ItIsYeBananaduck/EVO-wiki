---
title: MOC EVOconnect — Delegator
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/MOC EVOconnect — Delegator.md"]
updated: 2026-07-24
---

# MOC EVOconnect — Delegator
Delegator State Machine MOC
Purpose
Defines the canonical task lifecycle and authorization state machine enforced by the Delegator.
This MOC defines states and transitions only. All enforcement logic must conform to:

[Task Actionability Gate](https://app.notion.com/p/33ec72bad0138127a3cec9d764515869)
[Delegator Tool Hostage Rule](https://app.notion.com/p/33ec72bad013817d9b6bcc64f4a096fd)
[[Scoped Tool Grants]]
[Method Is Mandatory](https://www.notion.so/33ec72bad01381cb8c97fc4afc6ac233)
[Talent Promotion Rule](https://www.notion.so/33ec72bad0138181ae01de082f7c18fe)
[Talent Definition](https://www.notion.so/33ec72bad01381989ea8ca6805191903)
[Talent Revocation Rule](https://www.notion.so/33ec72bad0138175baa3d9c62607d0d6)
[Talent Definition](https://www.notion.so/33ec72bad013818095f9ecefbd4fea83)
[[EVOconnect — Delegator Talent Verification Doctrine]]
[[EVOconnect — Lightweight Talent Structure Addendum]]
[[EVOconnect — Skill Import and Conversion Doctrine]]
[[EVOconnect — Talent Backlink Navigation Doctrine]]

Entities
Task
id
title / goal
method_id (required)
talent_id (optional)
state (one of the states below)
run_count_successful (for method_id)
last_error (optional)
created_at / updated_at ### SwarmShard
shard_id
task_id / prompt_id
objective
allowed_inputs
output_schema
max_tokens / max_time
assigned_device_id
status: queued | running | completed | failed | expired
confidence (optional)
SwarmResultArtifact
shard_id
device_id
model_id (optional)
created_at
payload (structured per schema)
confidence - —
Swarm Overlay (Parallel Inference)
Purpose
Swarm is used when a prompt/inference is too compute-taxing for any single device.
Swarm does not change authorization rules. Swarm only distributes inference shards across Hive devices and merges results.

### Related:

[Swarm Parallel Inference](https://app.notion.com/p/33ec72bad013810eaa8ac7df235f10d0)
[Swarm Task Sharding](https://app.notion.com/p/33ec72bad013811e8b4ecb3b0ab21a89)
[Swarm Work Ticket](https://www.notion.so/Swarm-Work-Ticket-33ec72bad01381e29409d5ab37e5b618)
[Swarm Merge Rule](https://www.notion.so/Swarm-Merge-Rule-33ec72bad0138162ba34d5e3dcc576f1)
[Execution Lease Rule](https://www.notion.so/Execution-Lease-Rule-33ec72bad01381d7846bf9f2cabb72fd)

Swarm Invariants (Non-Negotiable)
Only the lease holder may initiate Swarm.
Swarm nodes execute inference-only work tickets unless explicitly allowed (default: no tools, no secrets).
Swarm results are advisory artifacts until the Delegator authorizes execution via:
method approval, or
Talent
Swarm cannot approve methods, promote Talents, or execute tools.

### Related:

[Task Actionability Gate](https://app.notion.com/p/33ec72bad0138127a3cec9d764515869)
[Delegator Tool Hostage Rule](https://app.notion.com/p/33ec72bad013817d9b6bcc64f4a096fd)
[[No Tool Access During Planning]]
[Secret Isolation Rule](https://www.notion.so/Secret-Isolation-Rule-33ec72bad013813e9eb6ead8af1141ad)

Swarm UI Rule
When Swarm is active, all participating device icons flash blue. The lease holder remains blue (and also flashes while participating).
Related: - [Hive Device Presence Icons](https://www.notion.so/Hive-Device-Presence-Icons-33ec72bad0138160b124e9d8d38126f0) - [Hive Icon Status Mapping](https://app.notion.com/p/33ec72bad01381848426f7f0f020919b)
Method
id
steps
required_tools
expected_outputs
risks (optional)
source: { standard | talent_trainer }
approved_history[] (timestamps + user_id)
successful_runs (counter)
promoted_to_talent_id (optional)
Talent
id
method_snapshot (immutable)
created_from_method_id
created_at
revoked_at

State Machine
State: Draft
Definition - Task exists but method is incomplete or missing required fields.
Rules - Tools: DENIED - Allowed actions: - edit task fields - define/modify method draft
Transitions - Draft → Planned (method complete)
Acceptance criteria - [ ] Task cannot leave Draft if method is missing required fields - [ ] No tools can be invoked in Draft
Overlay State: SwarmActive
Definition - Lease holder has activated Swarm to parallelize heavy inference into shards. - One or more Hive nodes are executing inference-only work tickets in parallel.
Rules - Tools: DENIED (Swarm work is inference-only by default) - Secrets: DENIED (no vault resolution on swarm nodes) - Allowed actions: - shard creation by lease holder - dispatch work tickets to nodes - receive partial artifacts - merge artifacts into a single result
Entry Conditions - Prompt/inference is estimated too heavy for any single device, OR - parallel execution reduces thermal/battery risk significantly, OR - lease holder lacks compute capability for the requested inference.
Exit Conditions - All shard tickets completed or expired, AND - Merge completed and merged artifact persisted.
Transitions - Any planning state → SwarmActive (lease holder activates Swarm) - SwarmActive → Planned (merged plan/method artifacts added) - SwarmActive → AwaitingApproval (if merged output includes a method proposal presented to user) - SwarmActive → Failed (if swarm fails to complete and no safe fallback exists)
Acceptance criteria - [ ] Swarm cannot grant tool access - [ ] Swarm nodes can only process scoped work tickets - [ ] All swarm outputs are schema-validated before merge - [ ] Swarm participation is visible via flashing blue device icons

State: Planned
Definition - Task has a complete method proposal, but no authorization path has been satisfied.
Rules - Tools: DENIED - Allowed actions: - present method to user - request approval - attach existing approved method reference - attach Talent reference (if exists)
Transitions - Planned → AwaitingApproval (method presented) - Planned → AwaitingTalent (task references talent_id but talent missing/revoked) - Planned → AuthorizedByTalent (task references a valid talent_id) - Planned → AuthorizedByMethod (user approves method)
Acceptance criteria - [ ] Delegator blocks all tools in Planned - [ ] A task referencing a Talent must validate that the Talent exists and is not revoked
Allowed actions - activate Swarm if inference is too heavy (produces planning artifacts only)

State: AwaitingApproval
Definition - Method has been presented; system is waiting for explicit user approval.
Rules - Tools: DENIED - Allowed actions: - user approves method - user rejects method (edit or cancel) - user requests changes
Transitions - AwaitingApproval → AuthorizedByMethod (user approves) - AwaitingApproval → Planned (user rejects or requests edits)
Acceptance criteria - [ ] No tool access until approval is recorded - [ ] Approval record must reference method_id + task_id + timestamp
Allowed Actions - activate Swarm if inference is too heavy (produces planning artifacts only)

State: AwaitingTalent
Definition - Task intends to execute via Talent but the referenced Talent is missing or revoked.
Rules - Tools: DENIED - Allowed actions: - user selects a different Talent - user chooses method approval instead - user recreates Talent (via trainer or promotion flow)
Transitions - AwaitingTalent → Planned (switch to method path) - AwaitingTalent → AuthorizedByTalent (valid talent selected)
Acceptance criteria - [ ] Task cannot execute while talent is missing/revoked - [ ] Delegator must not silently fall back to tools without approval

State: AuthorizedByMethod
Definition - User has approved the method for this task execution.
Rules - Tools: GRANTED (scoped) - ToolGrant must be issued: - allowed_tools = method.required_tools - scope bounded to task + method - TTL/max_steps applied
Transitions - AuthorizedByMethod → Executing (tool grant issued) - AuthorizedByMethod → Planned (approval revoked before execution)
Acceptance criteria - [ ] Tool grant is scoped (task_id, tools list, TTL, scope) - [ ] Revoking approval immediately invalidates tool grant

State: Authorized by Talent
Definition - Task references a valid Talent (immutable method snapshot) and may execute without per-task approval.
Rules - Tools: GRANTED (scoped to Talent method snapshot) - No method mutation allowed during execution.
Transitions - AuthorizedByTalent → Executing (tool grant issued) - AuthorizedByTalent → AwaitingTalent (Talent revoked before execution)
Acceptance criteria - [ ] Tool grant derives from immutable Talent snapshot, not mutable method - [ ] If Talent is revoked, execution cannot start

State: Executing
Definition - Task is actively running with tools enabled via a ToolGrant.
Rules - Tools: GRANTED only per ToolGrant - No scope expansion - No new tools beyond allowed_tools - Execution must be interruptible by user
Transitions - Executing → Completed (success) - Executing → Failed (failure) - Executing → Planned (user cancels; grant revoked)
Acceptance criteria - [ ] Any attempt to call a tool not in the grant is denied - [ ] Cancellation revokes grant immediately - [ ] Execution logs all tool calls with timestamps
Allowed Actions Swarm cannot be used to execute tools. Execution tool grants remain local to the lease holder.

State: Completed
Definition - Task succeeded and produced outputs.
Rules - Tools: DENIED (grant revoked/expired) - Must increment method.successful_runs if executed via method_id (including Talent-backed, increment underlying method lineage if desired) - Record completion artifact(s)
Transitions - Completed → EligibleForPromotion (if method meets threshold and not already promoted) - Completed → Archived (optional; for user organization) - Completed → Planned (if user reruns task)
Acceptance criteria - [ ] Successful completion creates durable artifact / log entry - [ ] successful_runs increments only on success Must create an audit record per: - [Task Audit Log Minimum Fields](https://www.notion.so/Task-Audit-Log-Minimum-Fields-33ec72bad01381aa9d87d0a77aa0cada) - [Task Transparency Retention](https://www.notion.so/Task-Transparency-Retention-33ec72bad013811ca9abdbbde305dc48)

State: Failed
Definition - Task execution did not complete successfully.
Rules - Tools: DENIED (grant revoked) - Provide safe error summary (no noisy interruptions unless needed)
Transitions - Failed → Planned (retry with edits) - Failed → AwaitingApproval (if method changed and needs re-approval) - Failed → Archived (optional)
Acceptance criteria - [ ] Tool grant revoked on failure - [ ] Failure reason recorded - [ ] No partial “success” counted toward promotion threshold unless explicitly defined Must create an audit record per: - [Task Audit Log Minimum Fields](https://www.notion.so/Task-Audit-Log-Minimum-Fields-33ec72bad01381aa9d87d0a77aa0cada) - [Task Transparency Retention](https://www.notion.so/Task-Transparency-Retention-33ec72bad013811ca9abdbbde305dc48)

State: EligibleForPromotion
Definition - A method has achieved the promotion threshold (e.g., 3 successful, re-approved runs) and is eligible to become a Talent.
Rules - Tools: DENIED - Eligibility does not auto-promote.
Transitions - EligibleForPromotion → PromotionProposed (Alice proposes promotion) - EligibleForPromotion → Completed (no action; user ignores) - EligibleForPromotion → Planned (if method changes; eligibility resets or re-evaluates)
Acceptance criteria - [ ] Eligibility requires N successful executions with explicit approvals (unless Talent Trainer path) - [ ] Eligibility alone does not grant tool access

State: PromotionProposed
Definition - Alice has proposed promoting the method to a Talent; awaiting explicit user consent.
Rules - Tools: DENIED - User must approve promotion explicitly.
Transitions - PromotionProposed → TalentCreated (user approves) - PromotionProposed → Completed (user declines; method remains reusable but not autonomous)
Acceptance criteria - [ ] Promotion consent is recorded separately from method approval - [ ] Declining promotion does not delete the approved method library entry

State: TalentCreated
Definition - A Talent has been created from the method.
Rules - Talent must snapshot the method immutably - No toggles; only revoke/delete
Transitions - TalentCreated → Planned (future tasks may reference Talent) - TalentCreated → TalentRevoked (if user revokes later)
Acceptance criteria - [ ] Talent snapshot cannot be edited - [ ] Talent appears as selectable execution path for future tasks

State: TalentRevoked
Definition - Talent no longer exists for execution purposes.
Rules - Any tasks referencing this talent become AwaitingTalent / Planned (non-actionable) until re-authorized.
Transitions - TalentRevoked → Planned (recreate method, re-approve) - TalentRevoked → TalentCreated (recreated via trainer or promotion)
Acceptance criteria - [ ] Revocation immediately prevents new executions via that Talent - [ ] No enable/disable toggle exists

Talent Trainer Overlay (Immediate Promotion Path)
Trainer Flow States (optional task subtype)
TrainerRecordingCaptured
TrainerMethodProposed
TrainerCoPilotedVerification
TrainerPromotionProposed
TalentCreated
Rules - Screen recording used to generate method proposal - Must include a co-piloted verification run - Upon user approval, method may be promoted immediately (bypasses N-run threshold)
Acceptance criteria - [ ] Trainer-created Talent requires supervised verification run - [ ] After verification + consent, Talent is created immediately - [ ] Recording is deletable and local by default (privacy boundary)

Global Delegator Invariants (Non-Negotiable)
Tools are denied unless state is AuthorizedByMethod or AuthorizedByTalent AND a valid ToolGrant exists
Planning states never have tools
No implicit autonomy: no auto execution without approval or Talent
No Talent toggles; only revoke/recreate
Talents are immutable snapshots

Suggested Test Checklist (Acceptance Suite)
Cannot execute task without approval or Talent
Reusing an approved method still requires explicit approval
After 3 successful approved executions, promotion becomes eligible
Promotion requires separate user consent
Trainer-created Talent can promote immediately only after co-piloted verification
Revoking a Talent prevents execution immediately
No UI/API exists to “disable” a Talent (only revoke/delete)
Tool calls outside the grant are denied and logged

Cross-System Coordination:
Delegator coordinates execution across domains and systems without introducing new intelligence.

It acts as the routing layer for multi-system workflows, ensuring tasks are directed to the appropriate execution context.

Delegator is responsible for:

- selecting the appropriate execution path
- routing tasks across systems and environments
- determining whether to execute, escalate, or defer
- coordinating multi-step and cross-domain workflows
^[{src_rel}]
