---
title: "EVOS1-284 — Connect Read Access to Training Cognition Without Data Ownership"
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
> Read-only contract for EVOconnect to access training cognition. `TrainingCognitionSummaryQuery` contract. Active schema contract.

# EVOS1-284 — Connect Read Access to Training Cognition Without Data Ownership

## Goal

Define a minimal, implementation-ready read path that allows **EVOconnect** to retrieve training cognition summaries while preserving doctrine boundaries:

- **EVOtraining owns logs and source events**
- **Cognition layer indexes and summarizes context**
- **EVOconnect reads and orchestrates, but does not own or mutate training data**

## Scope

### In scope

- Read-only query contract from Connect into cognition-facing training summaries.
- Ownership and mutation guardrails.
- Privacy/security boundaries for returned payloads.
- Query-path documentation for orchestration.

### Out of scope

- New training-log mutation APIs.
- Transfer of log ownership into Connect stores.
- Connect-side persistence of canonical training logs.

## Doctrine Alignment

This contract follows existing cognition and ownership doctrine in `/docs/evonotes/`:

- Domain ownership remains with source apps.
- Shared cognition is discoverability and interpretation infrastructure, not ownership transfer.
- Connect is orchestration/governance surface and must reference domain cognition instead of absorbing it.

## Contract: Read-Only Training Cognition Access

### Interface (conceptual)

```ts
type TrainingCognitionSummaryQuery = {
  user_id: string;
  from_utc?: string; // ISO-8601
  to_utc?: string;   // ISO-8601
  limit?: number;    // default 20, max 100
  include_evidence?: boolean; // default true
  next_cursor?: string; // pass previous response's next_cursor to fetch the next page; limit/max rules apply to each page
};

type TrainingCognitionSummary = {
  summary_id: string;
  domain: 'training';
  period: {
    from_utc: string;
    to_utc: string;
  };
  headline: string;
  key_signals: string[];
  confidence: number; // 0.0 - 1.0
  evidence_refs: Array<{
    source_type: 'workout_log' | 'set_log' | 'exercise_change' | 'intensity_record' | 'feedback';
    source_id: string;
    captured_at_utc?: string;
  }>;
  privacy_tier: 'pt1_user' | 'pt2_sensitive';
  generated_at_utc: string;
};

type TrainingCognitionSummaryResult = {
  items: TrainingCognitionSummary[];
  next_cursor?: string;
};
```

### Access method (conceptual)

```ts
getTrainingCognitionSummaries(query: TrainingCognitionSummaryQuery): Promise<TrainingCognitionSummaryResult>
```

### Hard invariants

1. **Read-only contract**: Connect can only call query/read methods for training cognition summaries.
2. **No log mutation**: No API in this path can create/update/delete training logs.
3. **No ownership transfer**: Returned data is derivative cognition context; canonical training records stay training-owned.
4. **Evidence-first summaries**: When `include_evidence` is true, returned summaries MUST include source references so Connect can explain provenance.
5. **Boundary-preserving storage**: Connect may cache transiently for request orchestration, but must not persist as canonical training data.

## Query Path (Documented)

1. User request or workflow intent enters **Connect orchestration layer**.
2. Connect resolves authorization scope for user/session and determines training context need.
3. Connect calls cognition read API for training summaries (`getTrainingCognitionSummaries`).
4. Cognition layer resolves indexed/derived summary objects linked to training-owned evidence.
5. Cognition layer returns read-only summary payload with evidence references and privacy tier.
6. Connect uses summaries to coordinate tasks, assistant responses, or cross-domain workflows.
7. Any action requiring training data mutation must route back to training-owned write paths (not this contract).

## Security and Privacy Boundaries

- **Least privilege**: Connect tokens/scopes for this path include read access only.
- **User boundary**: Queries are constrained to the requesting user identity and allowed tenancy/project space.
- **Tier-aware filtering**: Connect must honor `privacy_tier` and avoid exposing restricted fields on lower-trust surfaces.
- **Auditability**: Query operations should be logged as read events (actor, timestamp, scope, resource class).
- **No raw-log leakage by default**: Prefer summarized cognition output + evidence refs; raw event payload expansion requires explicit policy allowance.

## Acceptance Criteria Mapping

### 1) Connect retrieves training cognition summaries

Satisfied by the `getTrainingCognitionSummaries` read contract and typed summary result.

### 2) No data ownership violation

Satisfied by explicit invariants:

- read-only API shape,
- no mutation operations,
- no transfer of canonical ownership,
- boundary-preserving storage expectations.

### 3) Query path documented

Satisfied by the stepwise query-path section above from Connect orchestration to cognition read response and back into governed execution.

## Future Extension Hooks (Not required for EVOS1-284)

- Shared abstraction for future `mind` and `learn` summary queries with domain-specific evidence typing.
- Cross-domain aggregator query that composes summaries while preserving per-domain ownership metadata.
- Structured policy engine for field-level redaction based on privacy tier + surface capabilities.