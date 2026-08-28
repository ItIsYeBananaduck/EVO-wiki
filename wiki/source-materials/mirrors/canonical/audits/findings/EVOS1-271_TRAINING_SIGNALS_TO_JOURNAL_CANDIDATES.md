---
title: "EVOS1-271 — Map Existing Training Signals Into Journal Candidates"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-05-12
---

> **Archived — Promoted to Lifecycle System**
> - **Lifecycle stage**: spec
> - **Domain**: training
> - **Archival date**: 2026-05-12
> - **Archival reason**: Raw note classified and promoted to EVOnotes lifecycle system.
> - **Note**: Canonical/reference copy lives in docs/EVOnotes/spec/training/. This file is intake history only.

> **Status: Implementation Artifact**
> Maps EVOtraining signals to journal candidates. `TrainingJournalCandidate` contract. Active schema contract.

# EVOS1-271 — Map Existing Training Signals Into Journal Candidates

## Purpose

Define how existing EVOtraining signals are mapped into **journal candidates** for downstream cognition processing.

This issue defines mapping behavior only.

- ✅ In scope: signal-to-candidate categories, mapper rules, confidence/priority hints, boundary protection.
- ❌ Out of scope: auto-journal creation, journal persistence execution, signal pipeline rewrites.

---

## Boundary and Non-Goals

This mapping layer is **interpretation-ready metadata**, not journal creation.

- Mapper output is a `journal_candidate` shape for downstream consumers.
- Mapper MUST NOT write canonical `journal_entry` records directly.
- Mapper MUST NOT alter existing training signal producers.
- Existing ingestion/event pipelines remain source-of-truth and continue unchanged.

---

## Training Signal Families in Scope

1. Live set feedback signals (introduced in recent workout adaptation flows).
2. Workout diagnostic events.
3. Plan deviation signals.
4. Feedback pattern signals derived from repeated live feedback / adherence behavior.

---

## Candidate Category Contract (Training)

```ts
export type TrainingJournalCandidateCategory =
  | 'execution_quality'
  | 'fatigue_recovery'
  | 'exercise_preference'
  | 'load_tolerance'
  | 'adherence_risk'
  | 'plan_fit_adjustment'
  | 'safety_risk_signal'
  | 'coachability_feedback_pattern';

export interface TrainingJournalCandidate {
  candidate_id: string;
  user_id: string;
  category: TrainingJournalCandidateCategory;
  source_signal_type:
    | 'live_set_feedback'
    | 'workout_diagnostic_event'
    | 'plan_deviation'
    | 'feedback_pattern';
  source_refs: Array<{
    source_id: string;
    observed_at: string; // ISO-8601
  }>;
  summary_hint: string;
  confidence_hint: number; // 0..1 heuristic only
  priority_hint: 'low' | 'medium' | 'high';
  privacy_tier_hint: 'pt1_user' | 'pt2_sensitive' | 'pt3_restricted';
  auto_journal_create: false;
  metadata?: Record<string, unknown>;
}
```

**Hard requirement**: `auto_journal_create` is always `false` for EVOS1-271.

---

## Mapping Rules

Rules are evaluated top-down. A single signal may emit multiple candidates when justified (i.e., multiple separate `TrainingJournalCandidate` objects, each with a single `category` value in its `category` field).

## 1) Live Set Feedback → Journal Candidates

### Primary mappings

- perceived exertion unexpectedly high for prescribed load/rep target
  - `fatigue_recovery` (medium/high confidence based on repetition)
  - optional `load_tolerance`
- repeated "too easy" feedback with clean completion
  - `execution_quality`
  - `plan_fit_adjustment`
- repeated "pain/discomfort" feedback
  - `safety_risk_signal` (minimum `pt2_sensitive`, high priority)
- exercise-specific negative feedback while rest of session trends neutral/positive
  - `exercise_preference`

### Notes

- Single-instance outliers should stay lower confidence.
- Repeated confirmations across sessions raise confidence/priority in downstream scoring only.

## 2) Workout Diagnostic Events → Journal Candidates

Diagnostic events are operational-first, but selected training diagnostics can be candidate inputs.

### Mappings

- form-quality diagnostic degradation events
  - `execution_quality`
- persistent failure-to-complete-set diagnostics
  - `load_tolerance`
  - `fatigue_recovery`
- injury-risk/pain diagnostics
  - `safety_risk_signal` (high priority, `pt2_sensitive` minimum)
- benign transport/runtime diagnostics (network retry, sync lag, trace heartbeat)
  - **no candidate** (remain only event logs)

### Boundary with EVOS1-270

- `training.diagnostic.log` remains classified as `event_log` in ingestion.
- This mapping layer may read those events later to emit journal **candidates**, without changing ingestion classification.

## 3) Plan Deviations → Journal Candidates

### Mappings

- skipped planned session or repeated skipped accessories
  - `adherence_risk`
- frequent volume reductions vs plan target
  - `load_tolerance`
  - `plan_fit_adjustment`
- repeated exercise substitutions by user choice
  - `exercise_preference`
  - `plan_fit_adjustment`
- ad-hoc extra sets/workouts beyond plan with positive recovery signals
  - `execution_quality`
  - `plan_fit_adjustment`

## 4) Feedback Patterns → Journal Candidates

Pattern-level signals are aggregated across multiple underlying events.

### Mappings

- trend of corrective feedback followed by improved completion
  - `coachability_feedback_pattern`
  - `execution_quality`
- oscillating "too hard/too easy" across consecutive weeks
  - `plan_fit_adjustment`
- stable negative readiness + high RPE + missed targets pattern
  - `fatigue_recovery`
  - `adherence_risk`

---

## Suggested Heuristic Thresholds (Non-Binding)

- confidence baseline: 0.45 for single-signal candidate.
- confidence uplift: +0.10 to +0.20 for repeated corroboration in 7-14 day windows.
- priority escalation to `high` when:
  - category is `safety_risk_signal`, or
  - category is `adherence_risk` with repeated missed sessions.

These are mapper hints only and do not perform final journal acceptance.

---

## Pipeline Preservation Requirements

To preserve existing signal pipelines:

1. No mutation of existing event payload contracts.
2. No changes to producer-side event names.
3. No reclassification of ingestion targets already defined by EVOS1-270.
4. Mapper operates as a downstream consumer/adapter only.

---

## Implementation Notes

The **mapper** component referenced throughout this document transforms training signals from the event transport stream into `TrainingJournalCandidate` objects. Implementers should consider the following integration points:

### 1. Deployment Model

- **In-process within existing service**: Mapper runs as a module/handler within the current event ingestion or processing service, sharing the same runtime, memory, and error boundaries.
- **Standalone microservice**: Mapper runs as a separate service with independent scaling, deployment, and failure domains.

### 2. Trigger Mechanism

- **Event subscription to the transport stream**: Mapper subscribes to relevant training signal topics/channels on the event transport stream (e.g., Kafka, NATS, Supabase Realtime) and is invoked reactively whenever new signals are published.
- **Polling or scheduled batch**: Mapper queries persisted signal storage on a recurring schedule (e.g., cron, scheduled job) and processes accumulated signals in batches.

### 3. Execution Model

- **Synchronous inline processing**: Mapping executes immediately and blocks the caller until candidates are emitted (suitable for low-latency, single-event processing).
- **Asynchronous background worker**: Mapping is enqueued to a task queue (e.g., Bull, Temporal, RabbitMQ) and processed out-of-band by background workers.
- **Real-time streaming**: Mapper processes signals as a continuous stream with windowing logic for pattern detection (e.g., using stream processors or stateful event handlers).
- **Batch windowing**: Mapper accumulates signals over fixed time windows (e.g., 5 minutes, 1 hour) before running aggregation and mapping logic.

### 4. Event Transport Stream Subscription

When subscribing to the event transport stream:

- Use **consumer groups** or **topic subscriptions** to ensure scalable, load-balanced consumption of training signal events.
- Implement **retry policies with exponential backoff** for transient failures (e.g., downstream service unavailable).
- Ensure **idempotency**: duplicate signal processing must produce identical candidate outputs or be safely ignored (e.g., using `candidate_id` determinism or deduplication tables).
- Consider **ordering guarantees** if candidate emission order matters (e.g., per-user partition keys).

These choices directly affect latency, scalability, and operational complexity. Align decisions with existing event transport infrastructure and downstream candidate consumption patterns.

---

## Acceptance Criteria Mapping

- **All major training signals mapped**
  - live set feedback, workout diagnostics, plan deviations, and feedback patterns are mapped to explicit candidate categories.
- **No existing signal pipelines broken**
  - mapping is additive and downstream; producer and ingestion contracts remain unchanged.
- **Journal mapping defined but not auto-executed**
  - contract requires `auto_journal_create: false`; no direct journal persistence is introduced.