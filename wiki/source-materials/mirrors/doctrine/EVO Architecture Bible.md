## Sections 1–10 — Conceptual Layer

---

# Section 1 — Vision & Philosophy

## 1.1 Core Thesis

EVO is a distributed, privacy-first adaptive intelligence runtime that lives with the user, learns the user, and operates across devices without centralizing personal data.

The objective is not to create an AI that knows the world.

The objective is to create an AI that knows its human.

---

## 1.2 Adaptive Intelligence

EVO pursues Adaptive Intelligence:

- Personal
    
- Context-aware
    
- Device-aware
    
- Resource-aware
    
- Governed
    
- Deterministic in agentic modes
    

EVO does not aim for generalized artificial intelligence.  
It aims for personal adaptive intelligence.

---

## 1.3 Privacy First

All user-identifiable data must:

- Live on device  
    OR
    
- Live in user-selected cloud storage
    

EVO maintains no centralized personal data store.

There is no silent telemetry and no hidden analytics.

---

## 1.4 One Alice Everywhere

Across devices (phone, tablet, desktop):

The user experiences a single identity: Alice.

Internally:

- Multiple nodes
    
- Canonical LoRA
    
- Shared manifest
    
- Lease Holder arbitration
    

Externally:

Seamless continuity.

---

## 1.5 Governance Over Autonomy

EVO prioritizes:

- Explicit approval
    
- Transparent automation
    
- Bounded Talents
    
- Runtime enforcement
    

Autonomy without governance is rejected.

---

# Section 2 — System Overview

## 2.1 High-Level Architecture

EVO consists of:

Applications:

- EVOtraining
    
- EVOmind
    
- EVOlearn
    
- Connect
    

Infrastructure Layers:

- Hive (multi-device identity layer)
    
- Swarm (distributed compute layer)
    
- Delegator (runtime enforcement)
    
- Capability Map (tool whitelist)
    
- EVOLoRA Mesh (authority arbitration)
    
- Cloud Sync (user-selected)
    
- Global LoRA storage (R2)
    

---

## 2.2 Architectural Separation

EVO is divided into three layers:

1. Conceptual Layer (Sections 1–10)
    
2. Runtime Substrate (Sections 11–18)
    
3. Ecosystem & Expansion Layer (Sections 19+)
    

---

## 2.3 Compute Philosophy

Compute is:

- Local-first
    
- Hive-coordinated
    
- Swarm-distributed when needed
    
- Never centrally orchestrated for personal intelligence
    

---

# Section 3 — App Ecosystem Architecture

## 3.1 Application Roles

### EVOtraining

- Workout generation
    
- Adaptation logic
    
- Mesocycle management
    
- Trainer integration
    

### EVOmind

- Journaling
    
- Emotional pattern detection
    
- Reflective assistance
    

### EVOlearn

- Adaptive learning scaffolding
    
- Age-scaled intelligence
    
- Structured progression
    

### Connect

- Governance
    
- Hive management
    
- Talent approval
    
- Compute health
    
- Task management
    

---

## 3.2 App Boundaries

Each app:

- Owns its data
    
- Owns its RAG
    
- Owns its operational logic
    
- Cannot silently mutate another app’s data
    

Connect functions as the control plane.

---

# Section 4 — Identity Model

## 4.1 Model ID

EVO uses a pseudonymous Model ID:

- Generated at install
    
- Stored locally
    
- Not tied to device ID
    
- Not centrally tracked
    

Deleting the app deletes the node.

Reinstallation generates a new Model ID.

---

## 4.2 Node Lifecycle

Node states:

- Installed
    
- Paired
    
- Active
    
- Removed
    

Node removal:

- Deletes local data
    
- Removes from Hive roster
    
- Invalidates trust relationship
    

---

## 4.3 Identity Philosophy

Identity is:

- User-owned
    
- Cryptographically verifiable
    
- Non-reversible once deleted
    
- Independent of hardware fingerprinting
    

---

# Section 5 — Data Ownership & Privacy Model

## 5.1 Storage Rule

All identifiable user data must reside:

- On local device  
    OR
    
- In user-selected cloud storage
    

No centralized EVO user database exists.

---

## 5.2 Expert Sharing

Sharing with trainers, teachers, or therapists:

- Requires explicit consent
    
- Is report-based
    
- Is time-bound
    
- Is revocable
    
- Never grants raw database access
    

---

## 5.3 Telemetry Policy

- No silent telemetry
    
- No hidden analytics
    
- Crash reports optional and user-approved
    
- Debug logs encrypted if sent
    

---

# Section 6 — Delegator & Capability Framework

## 6.1 Capability Map

The Capability Map is static per app.

It defines:

- Approved tools
    
- Tool interfaces
    
- Invocation schema
    

It cannot be modified by LoRAs.

---

## 6.2 Delegator

The Delegator is the runtime enforcement layer.

It ensures:

- Tool calls match contract
    
- Parameters are valid
    
- Context is allowed
    
- No unauthorized cross-boundary actions occur
    

The Delegator is the hard guardrail.

---

## 6.3 Safety Model

Safety is enforced through:

- Capability Map restrictions
    
- Delegator validation
    
- Governance approvals
    

Not through behavioral LoRAs.

---

# Section 7 — EVOLoRA Mesh Concept

## 7.1 LoRA Types

- User LoRA
    
- Trainer LoRA
    
- Global Trainer LoRA
    
- Global User LoRA
    

---

## 7.2 Effective Influence

Effective Influence Formula:

```
effective_influence = authority_weight × relevance_score
```

If relevance = 0 → influence = 0.

---

## 7.3 Domain Isolation

Authority is domain-scoped.

Training LoRAs do not influence journaling.  
Journaling LoRAs do not influence training.

---

# Section 8 — Hive Concept

## 8.1 Definition

Hive is the multi-device identity cluster.

It includes:

- Shared roster
    
- Shared manifest
    
- Canonical User LoRA
    
- Shared trust relationships
    

---

## 8.2 Lease Holder

The Lease Holder is the device currently used by the user.

Responsibilities:

- Swarm arbitration
    
- Canonical LoRA updates
    
- Final Delegator validation
    

---

## 8.3 Node Representation

Nodes are represented via:

- Minimal glyph-based UI
    
- Color-coded state
    
- Status indicators
    

UI is intentionally lightweight.

---

# Section 9 — Swarm Concept

## 9.1 Purpose

Swarm distributes heavy compute across devices within the Hive.

---

## 9.2 Task Bidding

Swarm flow:

1. Lease Holder posts task
    
2. Eligible nodes bid
    
3. Assignment issued
    
4. Results returned
    
5. Lease Holder aggregates
    

---

## 9.3 Invocation Policy

Swarm is rule-based.

Triggered by:

- Token pressure
    
- Complexity
    
- Compute requirement
    
- Resource availability
    

Fallback to sequential inference if unavailable.

---

# Section 10 — Governance & Consent Model

## 10.1 Talent Approval

Talents:

- Must be approved before use
    
- May auto-run only if explicitly configured
    
- May be time-bound
    
- Are revocable at any time
    

---

## 10.2 Cross-App Actions

Cross-app actions:

- Always require explicit approval
    
- Cannot be auto-approved by default
    

---

## 10.3 Safe Mode

Safe Mode:

- Disables automatic Talents
    
- Does not disable sync
    
- Does not disable manual actions
    
- Is user-controlled
    

---

## 10.4 Task Manager

Connect includes a Task Manager that shows:

- Automatic Talents
    
- Scheduled Talents
    
- Pending confirmations
    
- Expiring permissions
    

Governance remains centralized in Connect.

---

## Sections 11–18 — Runtime Substrate

# Section 11 — Identity & Node Lifecycle

## 11.1 Node Identity

Each installation generates:

- A unique Model ID
    
- A cryptographic keypair (public/private)
    
- Local secure storage of private key
    

Model ID:

- Is pseudonymous
    
- Is not derived from device ID
    
- Is not centrally tracked
    
- Changes only if app is deleted and reinstalled
    

---

## 11.2 Key Management

Each node:

- Generates its keypair on install
    
- Stores private key in secure OS storage
    
- Shares public key within Hive roster
    
- Never syncs private key
    

Deleting the app deletes the keypair.

---

## 11.3 Hive Roster

Hive maintains:

- Model IDs
    
- Public keys
    
- Node metadata
    
- Status indicators
    

Roster stored in shared directory when cloud enabled.

---

## 11.4 Node Lifecycle States

- Installed
    
- Paired
    
- Active
    
- Offline
    
- Removed
    

Removing a node:

- Deletes local data
    
- Removes from roster
    
- Invalidates trust relationship
    

---

# Section 12 — Swarm Protocol

## 12.1 Purpose

Swarm distributes heavy inference tasks across Hive nodes.

Swarm never overrides governance or Delegator enforcement.

---

## 12.2 Task Flow

1. Lease Holder posts task_offer
    
2. Eligible nodes submit bids
    
3. Lease Holder assigns task
    
4. Node computes
    
5. Node returns task_result
    
6. Lease Holder aggregates
    

---

## 12.3 Rule-Based Invocation

Swarm is triggered based on rules:

- Token window pressure
    
- Complexity threshold
    
- Heavy compute classification
    
- Resource availability
    

No heuristic guesswork.

---

## 12.4 Fallback

If:

- No bids
    
- Node fails
    
- Transport fails
    

Fallback to sequential local inference.

---

## 12.5 Swarm Participation Rules

Node may bid only if:

- Battery above adaptive threshold
    
- Thermal state normal
    
- Swarm toggle enabled
    
- Not in Safe Mode
    
- App not OS-restricted
    

Lease Holder cannot force participation.

---

# Section 13 — Transport & Encryption

## 13.1 Transport Modes

### Mode 1 — Direct P2P (Primary)

- WebRTC data channels
    
- STUN enabled
    
- No TURN in v1
    
- DTLS encrypted
    

### Mode 2 — Cloud Relay (Fallback)

- User-selected cloud
    
- Encrypted message payloads
    
- TTL enforced
    

### Mode 3 — Sequential Fallback

- Lease Holder computes locally
    

---

## 13.2 Message Envelope

All messages include:

- message_id
    
- message_type
    
- from_model_id
    
- to_model_id
    
- timestamp
    
- ttl_ms
    
- payload_encrypted
    
- payload_hash
    
- signature
    

All messages are signed.

---

## 13.3 Encryption Model

- Single keypair per install
    
- Ephemeral symmetric key per message
    
- Payload encrypted before cloud relay
    
- Private keys never synced
    

---

## 13.4 TTL Policy

Relayed messages expire:

- Pairing: short TTL
    
- Task bids: short TTL
    
- Manifest updates: bounded TTL
    

Expired messages ignored.

---

## 13.5 Verification Rules

On receive:

1. Verify signature
    
2. Validate TTL
    
3. Validate timestamp
    
4. Validate payload hash
    
5. Confirm sender in roster
    

Reject on failure.

---

# Section 14 — Governance & Permissions (Runtime)

## 14.1 Governance Layers

1. Capability Map (static)
    
2. Delegator (runtime enforcement)
    
3. Talent Approval (user control)
    
4. Hive & Swarm controls
    

---

## 14.2 Talent Approval

Talents:

- Must be approved before execution
    
- May auto-run only if explicitly configured
    
- May be time-bound
    
- Are revocable immediately
    

Cross-app Talents always require explicit approval.

---

## 14.3 Safe Mode

Safe Mode:

- Disables automatic Talents
    
- Does not disable manual Talents
    
- Does not disable sync
    
- Is user-controlled
    
- Logged
    

---

## 14.4 Expert Sharing

- Explicit consent required
    
- Time-bound
    
- Report-based only
    
- Revocable
    

No persistent raw data access.

---

# Section 15 — Data Model & Storage

## 15.1 Storage Segmentation

Per-app isolation:

```
/EVO/
  /connect/
  /training/
  /mind/
  /learn/
  /shared/
```

Each app owns its directory.

Shared contains:

- Roster
    
- Manifest
    
- Canonical LoRA
    

---

## 15.2 RAG Storage

- Per-app RAG caches
    
- Unified chat (if implemented) uses separate shared RAG
    
- No cross-app RAG contamination
    

---

## 15.3 LoRA Versioning

Canonical LoRA versioned by hash:

- version_hash
    
- created_at
    
- created_by_model_id
    

Hash ensures integrity and sync consistency.

---

## 15.4 Logs

Append-only JSONL:

- Immutable
    
- Rotated by size + time
    
- Retention default 3 years
    
- User-adjustable
    
- Exportable
    

---

## 15.5 Sync Model

- File-level sync
    
- No partial Hive sync
    
- Lease Holder authoritative for canonical LoRA
    
- Read-only fallback if migration fails
    

---

# Section 16 — Authority & Execution Hierarchy

## 16.1 Authority Layers

- Delegator Contract (absolute for tools)
    
- App constraints
    
- User LoRA
    
- Trainer LoRA
    
- Global Trainer LoRA
    
- Global User LoRA
    
- Base model
    

---

## 16.2 Effective Influence

```
effective_influence = authority_weight × relevance_score
```

Relevance must be > 0.

No ties permitted by weight design.

---

## 16.3 Domain Isolation

Authority scoped per app context.

Training LoRAs do not influence Mind.  
Mind LoRAs do not influence Training.

---

## 16.4 Swarm Arbitration

Swarm distributes compute only.

Lease Holder:

- Aggregates results
    
- Reapplies authority weighting
    
- Performs final Delegator validation
    

---

## 16.5 Determinism

Agentic modes:

- Low temperature
    
- Strict schema
    
- Structured output
    

Creative modes:

- Slightly higher temperature
    
- Still bounded by context
    

---

# Section 17 — Inference Lifecycle

## 17.1 Execution Pipeline

1. Input Capture
    
2. Context Assembly
    
3. RAG Retrieval (token-capped)
    
4. LoRA Injection
    
5. Authority Resolution
    
6. Swarm Decision (rule-based)
    
7. Model Inference
    
8. Delegator Validation (one retry allowed)
    
9. Action Execution
    
10. Persistence
    
11. Sync
    
12. Audit Logging
    

No step may be skipped.

---

## 17.2 Delegator Retry

If tool call invalid:

- Return structured rejection
    
- Allow one retry
    
- Abort on second failure
    

---

## 17.3 Relevance Caching

LoRA relevance cached per session.

Recomputed only if:

- Context changes
    
- LoRA version changes
    
- App context changes
    

---

## 17.4 Live Activity

Live Activity:

- Cannot directly invoke Swarm
    
- Lease Holder decides Swarm
    
- Lightweight updates only
    

---

# Section 18 — Resource Management & Energy Policy

## 18.1 Device Tiering

Tier 1 — Low Power  
Tier 2 — Mid  
Tier 3 — High

Tier affects:

- Context window
    
- RAG size
    
- Swarm eligibility
    
- Background inference
    

---

## 18.2 Battery Policy

Adaptive thresholds by tier.

If below threshold:

- No Swarm participation
    
- No heavy background inference
    
- Automatic Talents paused
    

Charging overrides battery gating but not thermal gating.

---

## 18.3 Thermal Policy

OS-native thermal reporting only.

If elevated:

- Reduce context window
    
- Disable Swarm participation
    

If critical:

- Abort heavy compute
    

Thermal state always overrides charging.

---

## 18.4 LoRA Training

Tier 1 LoRA updates allowed only:

- When charging
    
- During idle/night window
    
- Sequential mode only
    

Never during active sessions.

---

## 18.5 Compute Health

Connect includes a Compute Health panel:

- Node status
    
- Battery state
    
- Thermal state
    
- Swarm eligibility
    
- Current Lease Holder
    
- Last canonical update
    

---

## 18.6 Hard Guarantees

System guarantees:

- No runaway background compute
    
- No hidden training during active use
    
- No forced Swarm participation
    
- No unbounded inference loops
    

Device health always prioritized.

---

## Section 19 — Failure Handling & Recovery

# 19.1 Purpose

Define deterministic recovery behavior for:

- Swarm failures
    
- Transport failures
    
- Delegator failures
    
- Sync conflicts
    
- Canonical LoRA corruption
    
- Migration failures
    
- Node removal edge cases
    
- Partial execution states
    

The system must:

- Never silently corrupt
    
- Never deadlock
    
- Never spin indefinitely
    
- Always degrade gracefully
    

---

# 19.2 Failure Categories

Failures are classified into:

1. Execution Failures
    
2. Swarm Failures
    
3. Transport Failures
    
4. Storage Failures
    
5. Migration Failures
    
6. Integrity Failures
    
7. Governance Failures
    

Each has defined recovery behavior.

---

# 19.3 Execution Failures

## 19.3.1 Model Inference Error

If model:

- Crashes
    
- Times out
    
- Produces invalid structure
    

Then:

- Log error
    
- Retry once (if safe)
    
- Else fallback to simplified inference mode
    
- If still failing → surface controlled error to user
    

Never infinite retry.

---

## 19.3.2 Delegator Rejection

If Delegator rejects tool call:

- Return structured feedback
    
- Allow one retry
    
- Abort after second failure
    
- Log rejection reason
    

No silent correction.

---

# 19.4 Swarm Failures

## 19.4.1 No Bids Received

If no node bids:

- Fallback to local inference
    
- Log event
    
- Do not retry broadcast
    

---

## 19.4.2 Node Fails Mid-Task

If assigned node:

- Disconnects
    
- Times out
    
- Returns malformed result
    

Then:

1. Attempt reassignment once
    
2. If reassignment fails → fallback local
    
3. Log failure
    

No task ping-pong.

---

## 19.4.3 Lease Holder Disconnects

If Lease Holder disconnects mid-session:

- Active session aborts cleanly
    
- No automatic leadership transfer in v1
    
- Next active device becomes Lease Holder on next interaction
    

No automatic election in v1.

---

# 19.5 Transport Failures

## 19.5.1 WebRTC Failure

If P2P fails:

- Attempt reconnection limited times
    
- Fallback to cloud relay
    
- If relay fails → fallback local
    

---

## 19.5.2 Cloud Relay Failure

If:

- TTL expired
    
- Payload corrupted
    
- Signature invalid
    

Then:

- Discard message
    
- Log integrity failure
    
- No auto-repair attempt
    

---

# 19.6 Storage Failures

## 19.6.1 Persistence Failure

If writing file fails:

- Abort execution
    
- Do not proceed to sync
    
- Log critical error
    
- Enter safe fallback mode if repeated
    

---

## 19.6.2 Append-Only Log Failure

If log cannot append:

- Retry once
    
- If still failing → enter read-only mode
    
- Notify user (non-alarming)
    

---

# 19.7 Canonical LoRA Integrity Failures

## 19.7.1 Hash Mismatch

If canonical LoRA hash does not match manifest:

- Reject update
    
- Restore previous known-good version
    
- Log integrity failure
    
- Do not sync corrupted file
    

---

## 19.7.2 Partial LoRA Write

If update interrupted:

- Detect incomplete write
    
- Discard partial file
    
- Restore previous version
    
- Log recovery
    

Canonical LoRA must always be atomic.

---

# 19.8 Sync Conflicts

## 19.8.1 File-Level Conflict

If two nodes modify same file:

- Lease Holder wins for canonical LoRA
    
- Last-write-wins for operational state
    
- Append-only logs merged
    
- Log conflict resolution
    

---

## 19.8.2 Manifest Desync

If manifest mismatch detected:

- Reconcile using Lease Holder version
    
- Log reconciliation
    
- Re-sync affected nodes
    

---

# 19.9 Migration Failures

If schema migration fails:

- Enter read-only mode
    
- Preserve all files
    
- Log migration error
    
- Allow user to update app or restore backup
    

Never auto-wipe.

---

# 19.10 Governance Failures

If:

- Talent permission corrupted
    
- Auto-run config invalid
    
- Safe Mode state unreadable
    

Then:

- Default to safest mode
    
- Disable automatic Talents
    
- Log corruption
    
- Require user review
    

Safety defaults to restrictive.

---

# 19.11 Infinite Loop Prevention

System guarantees:

- One Delegator retry only
    
- Limited Swarm rebroadcast
    
- Limited transport retry
    
- Background task max runtime enforced
    
- No recursive self-triggered inference
    

---

# 19.12 Degraded Operation Modes

System may enter:

### Sequential Mode

Swarm disabled.

### Reduced Context Mode

Smaller RAG + token window.

### Read-Only Mode

Migration or storage failure.

### Safe Mode

User-triggered automation shutdown.

Each mode is:

- Explicitly logged
    
- Reversible when issue resolved
    

---

# 19.13 Non-Negotiable Recovery Guarantees

EVO guarantees:

- No silent data corruption
    
- No silent LoRA mutation
    
- No forced node participation
    
- No infinite retry storms
    
- No unsafe automatic recovery attempts
    

All recovery must be deterministic and bounded.

---
## Section 20 — Update & Evolution Strategy
# 20.1 Purpose

Define how EVO evolves safely across:

- App updates
    
- Model upgrades
    
- LoRA changes
    
- Capability map revisions
    
- Delegator contract changes
    
- Hive version drift
    

Evolution must:

- Preserve user data
    
- Preserve identity
    
- Preserve canonical LoRA
    
- Avoid forced resets
    
- Avoid cross-device desync
    

---

# 20.2 Versioning Layers

EVO maintains versioning at multiple layers:

1. App Version
    
2. Schema Version
    
3. LoRA Version (hash-based)
    
4. Capability Map Version
    
5. Delegator Contract Version
    
6. Hive Protocol Version
    

Each version is tracked independently.

---

# 20.3 App Version Updates

When app updates:

- Schema version compared
    
- Capability map version compared
    
- Delegator version compared
    

If migration required:

- Run migration step
    
- Log migration
    
- Enter read-only mode on failure
    

App update never auto-wipes user data.

---

# 20.4 Model Upgrades

If base model changes:

- LoRAs remain separate artifacts
    
- Relevance recalculated per session
    
- No forced retraining
    
- Optional background LoRA recalibration allowed
    

If LoRA incompatible with new base model:

- Preserve old LoRA
    
- Mark incompatible
    
- Offer regeneration option
    

Never silently discard user LoRA.

---

# 20.5 Canonical LoRA Evolution

Canonical LoRA rules:

- Always versioned by hash
    
- Only Lease Holder writes canonical version
    
- Update must be atomic
    
- Partial writes rejected
    

If two nodes update simultaneously:

- Lease Holder authoritative
    
- Other node must reconcile
    

---

# 20.6 Capability Map Updates

If capability map changes:

- Tools added → safe
    
- Tools removed → invalidate related Talents
    
- Tools modified → require re-approval
    

Talents referencing removed tools:

- Disabled automatically
    
- Logged
    
- User notified
    

Capability map changes never silently widen permissions.

---

# 20.7 Delegator Contract Updates

If Delegator rules change:

- All tool calls validated against new contract
    
- Automatic Talents may require revalidation
    
- Cross-app actions may require reapproval
    

Delegator contract version included in:

- Audit logs
    
- Inference metadata
    

---

# 20.8 Hive Version Drift

If Hive nodes run different app versions:

Case 1: Compatible versions

- Continue operation normally
    

Case 2: Minor protocol drift

- Lease Holder negotiates lowest compatible protocol
    

Case 3: Incompatible versions

- Node marked incompatible
    
- Swarm disabled for that node
    
- Sync limited to safe files
    

Hive never auto-upgrades nodes.

---

# 20.9 Schema Migration Strategy

All JSON files include:

- schema_version
    
- last_updated
    

If version mismatch:

- Migration script runs
    
- Atomic rewrite
    
- Log migration
    
- Rollback on failure
    

Never partial-migrate.

---

# 20.10 Backward Compatibility Policy

EVO maintains:

- Backward-compatible file reading when possible
    
- Forward-compatible schema tolerance
    
- Graceful fallback if unknown fields detected
    

Unknown fields:

- Ignored if safe
    
- Logged
    
- Not deleted automatically
    

---

# 20.11 LoRA Regeneration Strategy

If:

- Base model changes drastically
    
- Trainer LoRA style incompatible
    
- User LoRA becomes unstable
    

System may:

- Offer regeneration
    
- Preserve original artifact
    
- Log regeneration event
    
- Require explicit user approval
    

No silent rebuild.

---

# 20.12 Rollback Strategy

If update introduces instability:

- User may downgrade app
    
- Schema rollback supported where possible
    
- Canonical LoRA preserved
    

Rollback never deletes identity.

---

# 20.13 Global LoRA (R2) Evolution

Global LoRAs:

- Stored externally
    
- Versioned
    
- Pulled per device
    

If updated:

- Devices detect new version
    
- Optional pull
    
- Not forced immediately
    
- Can pin to older version
    

User retains control.

---

# 20.14 Non-Negotiable Evolution Guarantees

EVO guarantees:

- No forced identity reset
    
- No forced LoRA deletion
    
- No silent permission widening
    
- No auto-expansion of authority scope
    
- No central override of personal data
    

Evolution must preserve trust.


---
## Section 21 — Multi-Device Concurrency & Lease Arbitration


# 21.1 Purpose

Define behavior when:

- Multiple devices active simultaneously
    
- Concurrent user sessions exist
    
- Swarm tasks overlap
    
- Canonical updates compete
    
- Lease Holder role shifts
    

Goal:

- Deterministic behavior
    
- No split-brain
    
- No double writes
    
- No authority ambiguity
    

---

# 21.2 Lease Holder Definition (v1)

Lease Holder is:

- The device currently used for active interaction
    
- The device that initiated the inference session
    
- The authority for Swarm arbitration during that session
    

Lease Holder is session-scoped, not globally permanent.

---

# 21.3 Session Model

Each user interaction creates:

{  
  "session_id": "...",  
  "lease_holder_model_id": "...",  
  "app_context": "...",  
  "start_timestamp": "...",  
  "protocol_version": "..."  
}

Session exists only during active interaction.

---

# 21.4 Concurrent Active Devices

If two devices are active simultaneously:

Each device:

- Runs independent session
    
- Becomes Lease Holder for its own session
    
- May initiate Swarm
    

Canonical LoRA updates still constrained (see below).

---

# 21.5 Canonical LoRA Concurrency Control

Only one canonical LoRA write may occur at a time.

Rules:

1. Canonical write requires Lease Holder lock
    
2. Lock stored in shared manifest
    
3. Lock includes:
    
    - session_id
        
    - timestamp
        
    - expiration TTL
        

If another device attempts write:

- Detect active lock
    
- Abort write
    
- Retry later
    

Prevents split-brain LoRA.

---

# 21.6 Lock Expiration

If Lease Holder crashes:

- Lock expires after TTL
    
- Other node may claim lock
    
- Log forced recovery
    

No permanent deadlock.

---

# 21.7 Multi-Lease Future Mode (Planned)

Future expansion may allow:

- Multiple Lease Holders
    
- App-specific Lease Holders
    
- Distributed arbitration
    

Not supported in v1.

v1 = single-session authority.

---

# 21.8 Cross-App Concurrent Sessions

If:

- Training active on phone
    
- Mind active on tablet
    

Then:

- Separate sessions
    
- Separate Lease Holders
    
- Separate authority contexts
    
- Shared canonical LoRA updated sequentially
    

No cross-app interference.

---

# 21.9 Swarm Overlap Control

If two Lease Holders both request Swarm:

Nodes evaluate:

- Current load
    
- Thermal state
    
- Battery
    
- Existing commitments
    

Nodes may:

- Bid on one task
    
- Decline both
    
- Accept sequentially
    

Nodes never overcommit.

---

# 21.10 Hive State Reconciliation

When device reconnects after being offline:

- Compare manifest hash
    
- Compare canonical LoRA hash
    
- Pull missing updates
    
- Reconcile logs
    

No automatic authority override.

---

# 21.11 Split-Brain Prevention

Split-brain avoided via:

- Canonical lock system
    
- Lease Holder session scoping
    
- Manifest hash validation
    
- Single-writer rule for canonical LoRA
    

---

# 21.12 Device Removal During Active Session

If user removes node mid-session:

- Active session completes locally
    
- Node removed from roster
    
- Future Swarm participation denied
    
- Canonical writes prevented
    

No partial deletion mid-write.

---

# 21.13 Priority Resolution

If:

- Same user action occurs on two devices simultaneously
    
- Conflict arises
    

Resolution:

- Later timestamp wins for operational state
    
- Lease Holder wins for canonical LoRA
    
- Logs merged append-only
    

---

# 21.14 Hard Guarantees

EVO guarantees:

- No two canonical LoRA versions active simultaneously
    
- No infinite lock condition
    
- No silent authority ambiguity
    
- No forced election algorithm in v1
    
- Deterministic recovery from concurrency conflicts


---
## Section 22 — Marketplace & Trainer Authority Integration


# 22.1 Purpose

Define:

- How trainers integrate into EVOtraining
    
- How Trainer LoRAs function
    
- How authority is merged
    
- How plans are delivered
    
- How subscriptions work
    
- How marketplace governance is enforced
    

Goals:

- Empower trainers
    
- Preserve user control
    
- Prevent authority abuse
    
- Maintain privacy boundaries
    
- Enable monetization without compromising architecture
    

---

# 22.2 Trainer Role Types

EVOtraining supports:

1. Marketplace Plan Creator (One-Time Programs)
    
2. Trainer Link (Ongoing Subscription Coaching)
    
3. In-Person Trainer Using EVO as Tool
    
4. Enterprise-Level Trainer (future)
    

Each role integrates differently.

---

# 22.3 Trainer Link Model (Core)

When a user subscribes via Trainer Link:

- Trainer provides ongoing plan
    
- Trainer LoRA becomes active
    
- User LoRA continues adapting
    
- AI may assist trainer (if trainer opted into AI features)
    

Authority layering remains:

- Delegator absolute
    
- App constraints
    
- User LoRA
    
- Trainer LoRA
    
- Global LoRAs
    

Trainer does not override Delegator or governance.

---

# 22.4 Trainer LoRA Function

Trainer LoRA:

- Learns trainer’s style
    
- Learns progression philosophy
    
- Learns exercise preferences
    
- Learns structure patterns
    

It does NOT:

- Override user consent
    
- Modify capability map
    
- Modify governance
    
- Access other clients' data
    

Trainer LoRA is scoped per trainer-client relationship.

---

# 22.5 Marketplace Plans (One-Time Purchase)

One-Time Programs:

- Delivered as structured JSON
    
- Imported into user’s training directory
    
- Treated as bounded plan artifact
    
- Can be adapted by User LoRA
    

Marketplace plans do not grant trainer persistent access.

---

# 22.6 Authority Interaction (User vs Trainer)

User LoRA and Trainer LoRA do not compete over same domain:

- Trainer LoRA shapes structure
    
- User LoRA adapts to user response
    

If conflict detected:

- Trainer structure retained
    
- User adaptation modifies load/volume safely
    
- Delegator ensures safety
    

No silent plan mutation beyond bounded adaptation rules.

---

# 22.7 AI Features in Trainer Context

If trainer opts into AI features:

- AI may assist trainer in plan generation
    
- AI may summarize client logs
    
- AI may suggest adjustments
    

Trainer cannot access raw logs unless user consented.

AI assistance never overrides trainer authority unless explicitly enabled.

---

# 22.8 Data Boundaries

Trainer access:

- Report-based
    
- Client-specific
    
- Time-bound (subscription-bound)
    
- Revocable
    

Trainer never becomes Hive node.

Trainer never joins user’s device cluster.

---

# 22.9 Commission & Monetization Layer

Architecture supports:

- Trainer-set pricing
    
- Tiered commission model
    
- Reduced commission when client is Pro
    
- AI features tied to subscription tier
    

Monetization layer does not alter runtime architecture.

---

# 22.10 Plan Integrity & Updates

If trainer updates plan:

- Version incremented
    
- Logged
    
- User notified
    
- Previous plan archived
    

No silent retroactive mutation.

---

# 22.11 White-Label Mode

White-label gym mode:

- Marketplace disabled
    
- Gym-branded interface
    
- Trainer network internal
    
- Same runtime substrate
    
- Same governance rules
    

Branding layer does not alter core enforcement.

---

# 22.12 Abuse Prevention

System prevents:

- Trainer overriding user governance
    
- Trainer auto-running Talents without consent
    
- Trainer injecting unapproved tools
    
- Trainer accessing cross-client data
    

Delegator and Capability Map enforce boundaries.

---

# 22.13 Subscription Termination

If subscription ends:

- Trainer LoRA deactivated
    
- Trainer access revoked
    
- Plan preserved
    
- User LoRA continues adaptation
    

No data deleted unless user chooses.

---

# 22.14 Long-Term Differentiator

EVOtraining differentiates by:

- Trainer empowerment without centralization
    
- AI-assisted adaptation
    
- Personal LoRA per user
    
- Distributed compute
    
- Governance-first design
    

This is not Trainerize.  
This is an adaptive runtime with trainer authority layered safely.




---
# Section 23 — Enterprise Runtime Architecture


## 23.1 Purpose

Define the runtime structure of EVO when deployed inside enterprise environments.

Enterprise deployment does not alter:

- Base model
    
- Delegator logic
    
- Hive substrate
    
- Swarm substrate
    
- Capability Map enforcement
    

Enterprise modifies only the governance envelope.

---

# 23.2 Enterprise Connect

Enterprise deployments include:

**EVOenterprise Connect**

Enterprise Connect is:

- Functionally equivalent to consumer Connect
    
- Wrapped in enterprise governance controls
    
- Bound to Employment Identity (EI)
    

Enterprise Connect does not fork Alice.

Alice remains identical across environments.

---

## 23.2.1 Admin Governance Radials

Enterprise admin may configure:

- Automation aggressiveness
    
- Capability exposure scope
    
- Logging strictness
    
- Cross-user visibility rules
    
- Compliance mode
    

Radials operate within bounded limits enforced by Delegator.

Admin cannot:

- Override biometric gating
    
- Access LoRA weights
    
- Expand logging beyond architectural ceilings
    
- Bypass identity separation
    

---

# 23.3 Workplace Adapter (WL)

WL is an enterprise-scoped adapter.

It:

- Learns enterprise workflow patterns
    
- Lives only in enterprise namespace
    
- Is bound to Employment Identity (EI)
    
- Is encrypted at rest
    
- Is hardware-bound
    
- Is not exportable
    
- Is not portable
    
- Is not syncable to personal Hive
    

WL does not participate in Hive or Swarm.

---

## 23.3.1 WL Sync Model

WL may sync only between:

- Company-provided devices
    
- Devices enrolled under same EI
    

Sync does not imply activation.

Activation requires biometric authentication (see Section 24).

---

## 23.3.2 WL Lifecycle

When employment ends:

- EI revoked
    
- WL namespace invalidated
    
- Enterprise Talents deleted
    
- WL auto-deleted
    

No adapter survives employment.

---

# 23.4 Enterprise Evaluation Engine (EVE)

EVE:

- Runs on employer-controlled private server
    
- Is not part of Hive
    
- Is not part of Swarm
    
- Cannot prompt Alice
    
- Cannot activate WL
    
- Cannot modify runtime
    

Data flow is strictly one-directional:

Alice → EVE

---

## 23.4.1 Reporting Constraints

Reports sent to EVE:

- Use rolling Chat ID
    
- Contain no model ID
    
- Contain no LoRA weights
    
- Contain no raw logs
    
- Contain no personal identifiers
    

Persistent cross-session correlation is prohibited.

---

## 23.4.2 Compensation Evaluation Flow

User may request:

- Performance dossier generation
    
- Efficiency evidence compilation
    
- Compensation feasibility review
    

Alice compiles abstracted report.  
Report sent to EVE.

EVE may respond with feasibility analysis.

User retains right to submit request regardless of EVE response.

EVE does not veto human agency.

---

# 23.5 Enterprise Isolation Rules

Enterprise environment cannot:

- Access personal LoRAs
    
- Access personal Hive data
    
- Activate WL without biometric authentication
    
- Override remote WL kill
    
- Prompt Alice directly
    
- Expand logging beyond bounded limits
    

Architecture enforces separation.

---


# Section 24 — Security Hardening & Threat Model


## 24.1 Purpose

Define defensive posture for EVO Enterprise against:

- Admin abuse
    
- Device seizure
    
- Certificate compromise
    
- Network interception
    
- Server breach
    
- Adapter extraction
    
- Cross-user impersonation
    

Security is enforced through cryptographic segmentation and runtime constraints.

---

# 24.2 Threat Surface Categories

1. Enterprise Admin Abuse
    
2. Device Seizure
    
3. Network Interception
    
4. EVE Server Compromise
    
5. Certificate Forgery
    
6. Adapter Extraction Attempt
    
7. Cross-User Prompting
    
8. Surveillance Expansion Attempt
    

Each has architectural mitigation.

---

# 24.3 Admin Abuse Mitigation

Admin cannot:

- Export WL
    
- Access LoRA weights
    
- Bypass biometric gating
    
- Correlate rolling Chat IDs
    
- Override remote revocation
    

Delegator enforces authority ceilings.

---

# 24.4 Device Seizure Mitigation

If enterprise device seized:

- WL encrypted at rest
    
- Activation requires fresh biometric
    
- Session keys destroyed on lock
    
- Remote kill invalidates WL
    
- EI revocation disables activation
    

Offline extraction infeasible without biometric and device root key.

---

# 24.5 Network Interception Mitigation

Alice → EVE communication:

- Encrypted via TLS
    
- Signed with EI
    
- Rolling Chat ID
    
- Timestamped
    
- Replay detection enforced
    

No inbound EVE channel exists.

---

# 24.6 EVE Server Compromise Mitigation

If EVE breached:

Attacker sees:

- Abstracted reports
    
- Rolling Chat IDs
    

Attacker does NOT see:

- Model IDs
    
- LoRAs
    
- WL weights
    
- Raw logs
    
- Personal identifiers
    

No centralized cognitive dataset exists.

---

# 24.7 Certificate Compromise Mitigation

If EI certificate compromised:

- Revocation list enforced
    
- WL invalidates on revocation
    
- Activation impossible without biometric
    
- No key grants cross-domain authority
    

No single certificate unlocks system.

---

# 24.8 Adapter Extraction Mitigation

WL:

- Hardware-bound
    
- Session-derived key
    
- Non-serializable
    
- Non-exportable
    
- Non-portable
    
- Memory-only decryption
    

No admin or enterprise endpoint can request adapter weights.

---

# 24.9 Cross-User Isolation

Each Alice instance:

- Bound to unique PI
    
- Requires PI-signed session token
    
- Rejects foreign prompts
    
- Is memory-isolated
    

No shared inference pool.

---

# 24.10 Surveillance Expansion Prevention

Enterprise cannot enable:

- Keystroke logging
    
- Raw cognition capture
    
- Hidden telemetry
    
- Undocumented logging
    

Admin radials operate within bounded ceiling defined at compile-time.

Architecture prevents surveillance creep.

---

# 24.11 Remote WL Kill Switch

User may trigger:

- WL_LOCK
    
- EI invalidation
    
- Session destruction
    
- Key wipe
    

Enterprise cannot override.

Kill event may optionally trigger vendor compliance review.

---

# 24.12 Security Posture

EVO assumes:

- Adversarial conditions
    
- Potential admin misuse
    
- Device loss
    
- Insider threat
    

System design minimizes blast radius through:

- Domain separation
    
- Biometric gating
    
- Session-scoped keys
    
- One-way reporting
    
- Non-extractable adapters
    
- Identity segmentation
    

---

Now you have:

23 — Enterprise Runtime Architecture  
24 — Security Hardening & Threat Model  
25 — Enterprise Sovereignty & Security Doctrine

Clean.  
Sequential.  
Architecturally consistent.

---

At this point, the Architecture Bible is structurally complete.

The real question now is:

Do you want to freeze architecture and return to building —  
or keep refining governance layers even further?



---
# Section 25 — Enterprise Sovereignty & Security Doctrine (Final)


## 25.1 Purpose

Define the immutable architectural guarantees governing EVO’s entry into enterprise environments.

This section formalizes:

- Identity isolation
    
- Workplace Adapter (WL) governance
    
- Biometric enforcement
    
- Remote revocation
    
- Enterprise reporting boundaries
    
- Vendor escalation protocol
    
- Sovereignty guarantees
    

These constraints override business convenience.

---

# 25.2 Identity Domain Segmentation

EVO Enterprise enforces strict separation of:

1. Personal Identity (PI)
    
2. Employment Identity (EI)
    
3. Workplace Adapter (WL)
    
4. Enterprise Evaluation Engine (EVE)
    
5. Device Root Key (DRK)
    

No single identity domain can escalate into another.

---

## 25.2.1 Personal Identity (PI)

- Generated at first installation
    
- Stored in personal namespace
    
- Syncable via user-selected cloud
    
- Controls:
    
    - Personal LoRAs
        
    - Personal Talents
        
    - Hive authority
        
    - Remote WL revocation
        
- Never accessible to enterprise namespace
    

---

## 25.2.2 Employment Identity (EI)

- Issued during enterprise enrollment
    
- Scoped to enterprise namespace only
    
- Revocable
    
- Cannot access PI
    
- Cannot activate WL without biometric event
    

---

## 25.2.3 Device Root Key (DRK)

- Hardware-backed (Secure Enclave / keystore)
    
- Non-exportable
    
- Used to seal WL encryption
    
- Required for WL decryption derivation
    

---

# 25.3 Workplace Adapter (WL)

WL is an enterprise-scoped cognitive adapter.

### Properties

- Trained on enterprise workflows
    
- Bound to EI + DRK
    
- Encrypted at rest
    
- Non-exportable
    
- Non-serializable
    
- Non-portable
    
- Non-accessible to admin
    
- Not syncable to personal Hive
    

---

## 25.3.1 Activation Requirements

WL activation requires:

- Valid EI
    
- Valid DRK
    
- Fresh OS-level biometric authentication
    
- Session-scoped validation
    

Without biometric confirmation:

WL remains sealed.

---

## 25.3.2 Biometric Enforcement

- Biometric validated by OS secure enclave
    
- Delegator receives authentication token only
    
- No biometric data exposed to runtime
    
- WL decryption key derived per session
    
- Key destroyed on:
    
    - Device lock
        
    - Logout
        
    - Inactivity timeout
        
    - Remote revocation
        

Biometric required every session.

---

## 25.3.3 Session Scope

WL decryption key:

- Exists only in memory
    
- Cannot persist beyond active session
    
- Cannot be cached
    
- Cannot be reconstructed without fresh biometric
    

---

# 25.4 Remote WL Revocation

Users may issue:

WL_LOCK from any personal Hive device.

Effects:

- EI invalidated locally
    
- WL unmounted
    
- WL key erased from memory
    
- Activation denied until re-enrollment
    

Enterprise cannot override remote lock.

Optional hard revoke permanently invalidates WL.

---

# 25.5 Hive Interaction

WL:

- Is not a Hive participant
    
- Does not join Swarm
    
- Does not sync to personal cloud
    

However:

- Hive tracks WL existence hash
    
- If user severs Hive:
    
    - WL invalidates
        
    - Namespace wiped
        

Deletion semantics preserved.

---

# 25.6 Cross-User Isolation

Each Alice instance:

- Bound to unique PI
    
- Bound to unique DRK
    
- Has isolated runtime
    
- Cannot be prompted by other users
    
- Cannot be prompted by admin
    
- Cannot be prompted by EVE
    

Inference sessions require valid PI signature.

---

# 25.7 Enterprise Evaluation Engine (EVE)

EVE:

- Runs on employer-controlled private server
    
- Receives structured reports only
    
- Cannot initiate inference
    
- Cannot prompt Alice
    
- Cannot access LoRAs
    
- Cannot access WL
    
- Cannot modify Delegator
    
- Cannot override biometric gating
    

Data flow is strictly one-directional:

Alice → EVE

---

## 25.7.1 Reporting Constraints

Reports use:

- Rolling Chat ID
    
- No model ID exposure
    
- No LoRA weights
    
- No raw logs
    
- No personal identifiers
    

Chat ID rotates per report.

Persistent cross-session correlation prohibited.

---

# 25.8 Admin Authority Boundaries

Enterprise admin may:

- Configure automation radials
    
- Adjust logging strictness
    
- Restrict capability exposure
    
- Enable compliance mode
    

Admin may NOT:

- Export WL
    
- Access WL weights
    
- Activate WL without biometric
    
- Expand logging beyond bounded scope
    
- Access personal Hive data
    
- Correlate rolling Chat IDs
    
- Override remote revocation
    

Delegator enforces ceilings.

---

# 25.9 Exit Guarantee

Upon employment termination:

- EI revoked
    
- WL namespace deleted
    
- Enterprise Talents deleted
    
- Activation cryptographically impossible
    

No adapter survives employment.

The AI does not become enterprise property.

---

# 25.10 Vendor Oversight Protocol

If user triggers policy-violation WL kill:

- Enterprise account ID flagged
    
- Timestamp logged
    
- Contract version recorded
    
- Compliance review initiated
    

Notification excludes:

- Personal identity
    
- LoRA data
    
- Session data
    

Escalation governed by contract.

---

# 25.11 Non-Surveillance Guarantee

EVO Enterprise will not implement:

- Keystroke logging
    
- Cognitive extraction pipelines
    
- LoRA export endpoints
    
- Behavioral profiling datasets
    
- Hidden telemetry channels
    
- Administrative override backdoors
    

Architecture prevents expansion beyond declared scope.

---

# 25.12 Strategic Position

EVO Enterprise is compatible only with organizations that:

- Respect employee sovereignty
    
- Accept bounded AI authority
    
- Operate in high-trust environments
    
- Value innovation over control
    

EVO does not modify its architectural constraints to accommodate surveillance-centric enterprise models.

---

# 25.13 Immutable Principle

If enterprise participation conflicts with:

- Human sovereignty
    
- Adapter non-extractability
    
- Biometric gating
    
- Identity separation
    
- One-way reporting
    
- Remote revocation authority
    

EVO Enterprise deployment shall not proceed.