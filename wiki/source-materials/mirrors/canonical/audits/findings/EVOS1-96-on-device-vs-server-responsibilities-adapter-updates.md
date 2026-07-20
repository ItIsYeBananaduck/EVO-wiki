---
type: audit-finding
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-96 Validation: On-Device vs Server-Side Responsibilities for Adapter Updates

Date: 2026-03-31  
Issue: EVOS1-96  
Scope: Validate and document which parts of adapter updates are executed on-device vs shared backend infrastructure, based on currently active implementation and agreed target pipeline direction.

References:

- `docs/audits/EVOS1-93-gu-gt-creation-storage-distribution-flow.md`
- `docs/audits/EVOS1-90-flyio-active-path-audit.md`
- `docs/audits/EVOS1-91-shared-federation-pipeline-global-adapters.md`
- `docs/audits/EVOS1-94-canonical-global-adapter-artifact-model-audit.md`

---

## Executive validation

The repository supports a clear responsibility boundary:

- **On-device:** local training-derived delta production, envelope signing/encryption, monotonic counter progression, artifact verification before apply, and local rollback safety behavior.
- **Server-side:** authenticated ingest, anti-replay enforcement, encrypted delta persistence, aggregation/evaluation/publication governance, and artifact metadata distribution.

Current production-like implementation is still a **single global patch lane** rather than separate GU/GT lanes. This does not change the boundary itself; it changes only how many artifact families the server publishes.

---

## Responsibility map by lifecycle stage

## 1) Update creation

### On-device responsibilities

- Produce local model deltas from private/user-local data.
- Prepare upload envelope fields (`delta_id`, `instance_key`, `model_version`, encrypted payload, checksum, size, counter).
- Encrypt payload material before transmission.
- Generate request signature used by federated upload validation.

### Server-side responsibilities

- Define and enforce upload contract (required fields, headers, versioning).
- Validate signature integrity and model-version/salt-based derivation rules.
- Reject malformed payloads and enforce payload size boundaries.

**Validation outcome:** responsibility split is implemented and consistent with current federated upload route behavior.

---

## 2) Transport and ingestion

### On-device responsibilities

- Transmit signed payload over network to federated endpoint.
- Preserve monotonic counter semantics and retry safely without counter regression.

### Server-side responsibilities

- Authenticate request and enforce anti-replay/counter checks.
- Store encrypted payload in object storage and bind it to metadata records.
- Persist immutable ingest metadata (timestamps, checksum/size/path/model version).
- Return deterministic conflict responses for replay/duplicate conditions.

**Validation outcome:** ingest trust boundary is server-enforced; device is responsible for producing valid monotonic submissions.

---

## 3) Aggregation and artifact generation

### On-device responsibilities

- None in active canonical path (no client-side merge orchestration).

### Server-side responsibilities

- Select eligible pending deltas for merge windows/batches.
- Decrypt and validate vector compatibility.
- Execute merge strategy (current implementation: averaged vector lane).
- Run safety/evaluation gate before publication.
- Produce merged artifact blob and compute checksum.

**Validation outcome:** aggregation is centralized on shared backend infrastructure.

---

## 4) Registry, versioning, and distribution

### On-device responsibilities

- Request latest artifact metadata from server endpoint.
- Verify checksum/signature policy before activation.
- Apply deterministic local activation ordering and retain last-known-good rollback state.

### Server-side responsibilities

- Persist publication metadata (version, checksum, publishedAt, download URL).
- Serve latest artifact metadata contract to clients.
- Govern artifact lifecycle transitions (publish/deprecate/revoke in target model).

**Validation outcome:** server is source of truth for global artifact metadata; device is consumer/verifier/applier.

---

## 5) Governance and policy execution

### On-device responsibilities

- Enforce local compatibility checks and activation policy gates tied to client/runtime capabilities.

### Server-side responsibilities

- Maintain policy and operational controls for release approval/rollback.
- Retain lineage evidence required for audits (batch linkage, checksums, timestamps, gate outcomes).
- Ensure only valid published artifacts are exposed as latest.

**Validation outcome:** governance controls are primarily server-owned, with device enforcing local runtime safety checks.

---

## RACI-style ownership matrix

| Lifecycle area                          | Device (R/A/C/I) | Server (R/A/C/I) | Notes                                                                  |
| --------------------------------------- | ---------------- | ---------------- | ---------------------------------------------------------------------- |
| Local training delta computation        | **R/A**          | I                | Data and training context stay local.                                  |
| Payload encryption + signing            | **R/A**          | C                | Server defines contract; device performs cryptographic packaging.      |
| Upload acceptance + anti-replay         | I                | **R/A**          | Counter replay controls are server-enforced.                           |
| Encrypted delta storage + metadata      | I                | **R/A**          | Shared infra owns persistence and traceability.                        |
| Merge orchestration + artifact creation | I                | **R/A**          | No active client merge path in canonical implementation.               |
| Safety/evaluation gate                  | I                | **R/A**          | Publish blocked unless server checks pass.                             |
| Artifact registry/versioning            | I                | **R/A**          | Single-lane now; typed-family target defined in EVOS1-91.              |
| Artifact fetch + verification + apply   | **R/A**          | C                | Device verifies and activates; server supplies metadata and blob URL.  |
| Rollback execution (runtime)            | **R/A**          | C                | Device runtime rollback; server governs published rollback candidates. |
| Release governance and audit evidence   | I                | **R/A**          | Governance remains centralized.                                        |

Legend: R = Responsible, A = Accountable, C = Consulted, I = Informed.

---

## Current-state caveat: single-lane artifact model

Per EVOS1-94 and EVOS1-93, active implementation publishes one `global_patch` lane. Therefore:

- On-device logic should currently assume one global lane unless explicitly migrated.
- Server-side ownership already fits future GU/GT split; only artifact typing/publication lanes need extension.

This means EVOS1-96 does **not** require moving merge logic to devices; it requires preserving the boundary while introducing typed family lanes when GU/GT becomes first-class.

---

## Decision log for EVOS1-96

1. **Boundary validated:** Device performs local compute + secure submission + local verification/apply; server performs shared ingest/merge/publish governance.
2. **No boundary inversion needed:** Aggregation remains server-side in both current and target models.
3. **Future GU/GT implementation impact:** Mostly schema/API/pipeline typing changes on server, plus device retrieval policy updates; not a shift of core merge responsibility to device.

---

## Acceptance mapping (EVOS1-96)

- Validate which parts happen on device vs server: **Completed** in lifecycle map + matrix.
- Align with current implemented reality: **Completed** (single-lane global patch noted explicitly).
- Keep compatibility with target shared pipeline definition: **Completed** (typed-family extension path retained).

---

## Recommended follow-up implementation checklist

1. Add typed artifact family discriminator in metadata schema/API (GU/GT capable) while preserving backward compatibility.
2. Add explicit client runtime policy for family selection and fallback.
3. Preserve current anti-replay and signature validation gates unchanged across migration.
4. Document rollback ownership split (server-published candidates, device-executed fallback) in runtime runbooks.