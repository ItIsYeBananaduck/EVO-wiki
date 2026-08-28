---
title: "EVOS1-270 — Log Ingestion Scaffold and Event Boundaries"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-05-12
---

> **Archived — Promoted to Lifecycle System**
> - **Lifecycle stage**: spec
> - **Domain**: connect
> - **Archival date**: 2026-05-12
> - **Archival reason**: Raw note classified and promoted to EVOnotes lifecycle system.
> - **Note**: Canonical/reference copy lives in docs/EVOnotes/spec/connect/. This file is intake history only.

> **Status: Implementation Artifact**
> Event-layer scaffold for cross-app log ingestion. `LogIngestionEnvelope` contract. Active schema contract.

# EVOS1-270 — Log Ingestion Scaffold and Event Boundaries

## Purpose

Define the **event-layer scaffold** for cross-app log ingestion across EVOtraining, EVOmind, EVOlearn, and EVOconnect.

This document intentionally defines contracts, boundaries, and routing/classification behavior only.

- ✅ In scope: schemas, ingestion interface, classification targets, event-vs-observation boundary.
- ❌ Out of scope: full ingestion pipeline implementation, storage implementation, journal synthesis implementation.

---

## Layer Boundary (Non-Negotiable)

The ingestion layer handles **raw/near-raw events** and emits classified ingestion envelopes.

It does **not**:
- generate canonical `journal_entry` records,
- synthesize user-learning summaries,
- perform long-horizon cognition inference.

Those capabilities remain in downstream cognition/journal components (see EVOS1-269 contract).

---

## Canonical Ingestion Interface (Scaffold)

### TypeScript Contract

```ts
export type AppDomain = 'EVOtraining' | 'EVOmind' | 'EVOlearn' | 'EVOconnect';

export type ClassificationTarget = 'event_log' | 'observation_candidate' | 'ignore';

export interface LogIngestionEnvelope<TPayload = Record<string, unknown>> {
  envelope_id: string;
  user_id: string;
  app_domain: AppDomain;
  event_name: string;
  occurred_at: string; // ISO-8601 date-time
  received_at: string; // ISO-8601 date-time
  source: {
    platform?: 'ios' | 'android' | 'web' | 'backend' | 'unknown';
    app_version?: string;
    device_id?: string;
    session_id?: string;
    trace_id?: string;
  };
  payload: TPayload; // app-native event payload
  privacy_tier?: 'pt0_public' | 'pt1_user' | 'pt2_sensitive' | 'pt3_restricted';
  metadata?: Record<string, unknown>;
}

export interface IngestionDecision {
  target: ClassificationTarget;
  reason: string;
  observation_hint?: {
    observation_type: string;
    confidence_hint?: number; // 0..1, optional heuristic only
  };
}

export interface LogIngestionRouter {
  classify<TPayload>(envelope: LogIngestionEnvelope<TPayload>): IngestionDecision;
}
```

### Behavioral Notes

- `event_log`: persist/process as event evidence; no journal write.
- `observation_candidate`: keep as event evidence and mark eligible for later interpretation.
- `ignore`: drop from cognition ingestion path (may still exist in operational telemetry elsewhere).

---

## Per-App Event Schemas (Scaffold)

> These are boundary schemas for ingestion classification, not full app analytics schemas.

## 1) EVOtraining Event Schema

```json
{
  "$id": "evo://contracts/ingestion/evotraining.event.schema.json",
  "type": "object",
  "required": ["event_name", "occurred_at", "payload"],
  "properties": {
    "event_name": {
      "enum": [
        "training.session.started",
        "training.session.completed",
        "training.exercise.completed",
        "training.plan.adjusted",
        "training.diagnostic.log"
      ]
    },
    "payload": {
      "type": "object",
      "properties": {
        "session_id": { "type": "string" },
        "exercise_id": { "type": "string" },
        "diagnostic_type": { "type": "string" },
        "severity": { "enum": ["debug", "info", "warn", "error"] }
      },
      "additionalProperties": true
    }
  },
  "additionalProperties": true
}
```

**Training diagnostic log rule**: `training.diagnostic.log` is always classified as `event_log`.

## 2) EVOmind Event Schema

```json
{
  "$id": "evo://contracts/ingestion/evomind.event.schema.json",
  "type": "object",
  "required": ["event_name", "occurred_at", "payload"],
  "properties": {
    "event_name": {
      "enum": [
        "mind.checkin.submitted",
        "mind.reflection.captured",
        "mind.exercise.completed",
        "mind.diagnostic.log"
      ]
    },
    "payload": { "type": "object", "additionalProperties": true }
  },
  "additionalProperties": true
}
```

## 3) EVOlearn Event Schema

```json
{
  "$id": "evo://contracts/ingestion/evolearn.event.schema.json",
  "type": "object",
  "required": ["event_name", "occurred_at", "payload"],
  "properties": {
    "event_name": {
      "enum": [
        "learn.lesson.started",
        "learn.lesson.completed",
        "learn.quiz.submitted",
        "learn.diagnostic.log"
      ]
    },
    "payload": { "type": "object", "additionalProperties": true }
  },
  "additionalProperties": true
}
```

## 4) EVOconnect Event Schema

```json
{
  "$id": "evo://contracts/ingestion/evoconnect.event.schema.json",
  "type": "object",
  "required": ["event_name", "occurred_at", "payload"],
  "properties": {
    "event_name": {
      "enum": [
        "connect.checkin.sent",
        "connect.checkin.received",
        "connect.group.activity",
        "connect.diagnostic.log"
      ]
    },
    "payload": { "type": "object", "additionalProperties": true }
  },
  "additionalProperties": true
}
```

---

## Routing Rules (Scaffold)

Rules are evaluated top-down; first match wins.

1. **Operational diagnostics**
   - `*.diagnostic.log` → `event_log`
   - Rationale: useful trace evidence, not interpreted cognition.

2. **Domain behavior events**
   - session completion, check-ins, lesson completion, group activity → `observation_candidate`
   - Rationale: these may support future interpretation, but ingestion does not interpret.

3. **Pure transport/noise events**
   - heartbeat, retry, ack-only transport events → `ignore`
   - Rationale: no cognition value for event layer.

4. **Unknown event names**
   - default `event_log` with reason `unmapped_event_defaulted_to_event_log`
   - Rationale: fail-open for auditability.

---

## Raw Event vs Interpreted Observation Boundary

**Raw event (ingestion-owned):**
- directly emitted by app/backend systems,
- preserves source payload and provenance,
- can be classified but not semantically narrated.

**Interpreted observation (downstream-owned):**
- derived statement about the user, behavior, or pattern,
- may include confidence and conflict handling,
- can eventually contribute to journal generation.

### Explicit Boundary Rule

Ingestion may set `observation_hint` metadata only. It MUST NOT create persistent observation objects and MUST NOT create journal entries.

---

## Non-Goals for EVOS1-270

- No ingestion workers/queues implemented.
- No storage migrations introduced for full pipeline.
- No automatic journal generation.
- No cross-app summarization/inference logic.

---

## Acceptance Criteria Mapping

- **Log ingestion interface is defined** → `LogIngestionEnvelope`, `IngestionDecision`, `LogIngestionRouter`.
- **Per-app log schemas are defined** → four app-specific scaffold schemas.
- **Routing rules are defined** → ordered classification rule set above.
- **Training diagnostic logs are classified as event logs** → explicit `training.diagnostic.log` rule.
- **No automatic journal generation occurs** → boundary + non-goals explicitly prohibit journal writes.