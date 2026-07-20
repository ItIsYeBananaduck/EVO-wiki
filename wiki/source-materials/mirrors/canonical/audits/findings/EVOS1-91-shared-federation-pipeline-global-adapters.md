---
type: audit-finding
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-91 Definition: Shared Federation Pipeline for Global Adapters

Date: 2026-03-31  
Issue: EVOS1-91  
Scope: Define the canonical, shared system for creating, storing, versioning, and distributing **global adapters** across domains, using GU and GT as initial concrete families without hardcoding Training-only semantics.

References:

- `docs/audits/EVOS1-90-flyio-active-path-audit.md`
- `docs/audits/EVOS1-93-gu-gt-creation-storage-distribution-flow.md`
- `docs/audits/EVOS1-94-canonical-global-adapter-artifact-model-audit.md`

---

## Executive definition

The canonical federation system should be a **typed global artifact pipeline** with:

1. A domain-agnostic artifact contract (`artifact_family`, `domain`, `base_model`, `version`, `checksum`, `storage_uri`, `published_at`, `status`).
2. Shared ingestion and aggregation infrastructure across all domains.
3. Family-specific aggregation jobs (GU/GT now; extensible later).
4. Single governance control plane for approval/promote/rollback.
5. Uniform client retrieval contract with deterministic selection rules.

This design preserves current working merge/upload foundations while introducing explicit typed publication lanes required for GU/GT and future global adapters.

---

## Current truth (baseline from audits)

- Active executable pipeline today is a **single global patch lane** (`global_patch.bin` + `global_patch_metadata`).
- Upload, merge, and publish flows are implemented and service-hosting agnostic.
- GU/GT concepts are present in docs but not first-class in schema/API execution.

Therefore, EVOS1-91 defines the **target canonical shared pipeline contract** to evolve from the current single-lane implementation.

---

## Canonical shared pipeline (target model)

## 1) Artifact identity model (domain-agnostic)

Each published global artifact must include:

- `artifact_id` (immutable UUID)
- `artifact_family` (enum; initial values: `GU`, `GT`)
- `domain` (string; e.g., `training`, `nutrition`, `recovery`)
- `base_model_id` (string)
- `base_model_version` (string)
- `aggregation_window_start` / `aggregation_window_end`
- `version` (monotonic, scoped by `(domain, artifact_family)`)
- `checksum_sha256`
- `signature` (optional initially, mandatory at hardening milestone)
- `storage_uri`
- `created_at`, `published_at`
- `status` (`draft`, `validated`, `published`, `deprecated`, `revoked`)
- `lineage` (inputs: batch ids, policy version, eval report id)

### Why this is canonical

- Makes GU/GT explicit but avoids training-specific lock-in through `artifact_family + domain`.
- Supports future families without schema rewrites.
- Enables governance/auditability and deterministic rollback.

## 2) Shared ingestion plane

All domains submit encrypted deltas through one common ingestion contract:

- Authenticated, signed envelope upload.
- Counter/replay protection.
- Payload integrity checks.
- Domain + candidate family routing metadata captured on ingest.

Shared components:

- API gateway + signature validation
- Delta metadata table
- Blob/object store
- Replay/counter enforcement

## 3) Shared aggregation orchestration + family-specific jobs

A common scheduler/orchestrator controls aggregation windows and executes family jobs:

- Step A: Resolve eligible deltas for `(domain, artifact_family, window)`.
- Step B: Perform merge strategy configured per family (mean, weighted mean, robust aggregation).
- Step C: Run quality/safety evaluation gates.
- Step D: Package artifact + metadata manifest.
- Step E: Stage for approval.

GU and GT are first implemented as two separate jobs under the same orchestration framework.

## 4) Shared registry and versioning

Replace single-lane registry semantics with typed registry semantics:

- One registry table or service with keys: `(domain, artifact_family, version)`.
- Unique active published version per `(domain, artifact_family)`.
- Explicit lifecycle transitions (`draft -> validated -> published`).
- Immutable metadata for published versions.
- Rollback pointer tracked per `(domain, artifact_family)`.

## 5) Shared distribution contract

Client/API retrieval should be family-aware but uniform:

- `GET /api/v1/federated/artifacts/latest?domain=<d>&family=<f>`
- `GET /api/v1/federated/artifacts?domain=<d>&family=<f>&limit=<n>`
- `GET /api/v1/federated/artifacts/<artifact_id>`

Response includes:

- version, checksum, signature, storage URL (or signed URL), publishedAt, deprecation status, minimum compatible client/runtime.

This keeps delivery generic across GU/GT and future adapter families.

---

## On-device vs server responsibilities (canonical split)

## Device responsibilities

- Generate local delta updates from private on-device data.
- Encrypt/sign upload envelope with monotonic counters.
- Fetch latest artifact metadata for required families.
- Verify checksum/signature before activation.
- Apply deterministic adapter ordering rules locally.
- Keep last-known-good artifact for rollback.

## Server responsibilities

- Validate upload authenticity/integrity and enforce anti-replay.
- Store encrypted deltas and lineage metadata.
- Run aggregation and evaluation pipelines.
- Produce typed artifacts (GU/GT now, extensible families later).
- Govern lifecycle transitions (approve/publish/revoke/rollback).
- Serve typed artifact metadata/distribution endpoints.

This split keeps private data local while centralizing global artifact governance.

---

## Canonical rollout plan (incremental)

## Phase 1 — Contract introduction (no client break)

- Add typed artifact metadata model in schema/API while continuing to publish existing single global patch.
- Introduce compatibility mapping: legacy `global_patch` -> `artifact_family=GU` for deterministic migration.
- Add read APIs for typed latest retrieval.

## Phase 2 — Dual-family publication

- Run two publication jobs: GU and GT.
- Persist independent versions and checksums per family.
- Keep legacy endpoint as compatibility proxy until clients migrate.

## Phase 3 — Client family-aware consumption

- Update client resolver to request required families explicitly.
- Apply policy-based enablement/gating for GU/GT use.
- Enforce signature verification + rollback policy.

## Phase 4 — Legacy lane retirement

- Deprecate single-lane `global_patch_metadata` contract.
- Make typed registry the only source of truth.
- Remove ambiguous docs and stale wiring.

---

## Governance rules for shared pipeline

1. **No untyped global publication** once Phase 2 is active.
2. Every published artifact must have lineage + evaluation evidence.
3. Family versions are monotonic and immutable once published.
4. Rollback must be possible without recomputing historical merges.
5. Domain/family ownership must be explicit (approver groups, policy versions).

---

## Direct answers for EVOS1-91 acceptance intent

- **Shared federation pipeline defined?** Yes — defined as a typed, domain-agnostic pipeline with shared ingest/orchestration/registry/distribution layers and family-specific jobs.
- **GU/GT included as first concrete cases?** Yes — GU/GT are first-class initial `artifact_family` values.
- **Hardcoded to Training semantics?** No — domain is explicit and independent from family.
- **On-device vs server responsibilities clarified?** Yes — split is defined above.
- **Aligned with current implementation reality?** Yes — rollout is incremental from the current single-lane `global_patch` baseline.

---

## Implementation handoff to sibling issues

- EVOS1-95: infrastructure/runtime migration decisions should not alter typed artifact contract.
- EVOS1-96: finalize runtime gating matrix for family activation rules and fallback behavior.
- Future execution task: implement schema/API migration from single-lane metadata to typed global artifact registry.