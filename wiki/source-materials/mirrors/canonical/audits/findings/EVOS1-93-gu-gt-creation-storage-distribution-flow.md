---
type: audit-finding
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-93 Audit: Current GU/GT Creation, Storage, and Distribution Flow

Date: 2026-03-31
Scope: Trace the _current implemented path_ from local training output through aggregation, packaging, upload, storage, registry, and delivery for GU/GT-class global artifacts.
Reference: `docs/audits/EVOS1-90-flyio-active-path-audit.md`

## Executive conclusion

- The active production-like backend path currently produces and distributes a **single global patch artifact** (from aggregated encrypted deltas), not distinct GU and GT adapter artifacts.
- GU/GT terminology appears broadly in planning/spec docs, but code-backed execution currently converges into `global_patch_metadata` + `learning-deltas/global-patches/.../global_patch.bin`.
- There are **two active pipelines** in repo:
  1. FastAPI + Supabase federated path (`federated-server/src/...`) → canonical for encrypted upload + merge + patch metadata.
  2. Node staging path (`api/staging/...`) → local file-backed aggregation + patch publishing for staging/ops/testing.
- Therefore, for EVOS1-93, the “current GU/GT flow” is best described as:
  - **Implemented:** shared global patch federation flow.
  - **Not yet implemented separately:** explicit GU artifact channel and explicit GT artifact channel.

---

## Definitions used in this audit

- **GU/GT (expected model):** Global User adapter / Global Trainer adapter as separate global artifacts.
- **Current implemented artifact:** global patch vector/blob published via `global_patch_metadata`.

---

## End-to-end trace (implemented path)

## 1) Local training output and queued upload payload

### Client-side output shape (as accepted by server)

The canonical federated ingest endpoint accepts encrypted payload envelopes containing:

- `delta_id`
- `instance_key`
- `model_version`
- `encrypted_payload` (base64)
- `salt`
- `checksum`
- `size_bytes`
- monotonic `counter`
- optional training metadata

Endpoint:

- `POST /api/v1/federated/upload-delta`
- Router implementation: `federated-server/src/api/upload_delta.py`

Security/validation gates applied at ingest:

- feature-flag gate `fl.staging_upload`
- required `X-Federated-Signature` and version `1`
- HMAC verification derived from `model_version + salt`
- payload size validation and replay/counter controls through RPC

## 2) Upload and storage of encrypted deltas

After request validation, encrypted payload bytes are written to Supabase Storage:

- bucket: `learning-deltas`
- path pattern: `{instance_key}/{received_at}/{delta_id}.enc`

Metadata is then recorded via Supabase RPC:

- RPC: `record_federated_delta`
- table destination (through RPC internals): `encrypted_delta_records`

Stored metadata includes:

- model version
- payload checksum/size
- salt
- counter
- created/received timestamps
- storage path linkage

Anti-replay semantics are enforced via error mapping:

- replay/duplicate/counter regressions return HTTP `409`.

## 3) Aggregation trigger and batch orchestration

Merge/aggregation entrypoints:

- `POST /api/v1/aggregate/trigger`
- `POST /api/v1/federated/merge`

Both call `merge_deltas(job_id)` in:

- `federated-server/src/ml/gguf_merger.py`

Batch orchestration steps:

1. Select pending rows from `encrypted_delta_records` where `merge_batch_id IS NULL`.
2. Create `merge_batches` row with status `MERGING`.
3. For each selected row:
   - fetch encrypted blob from `learning-deltas`
   - derive decrypt key from `salt + model_version`
   - AES-GCM decrypt
   - parse float32 delta vector
4. Exclude incompatible/malformed vectors.

## 4) Global artifact creation (current stand-in for GU/GT)

Merged output is computed as element-wise mean across accepted vectors.

Safety gate:

- `run_safety_evaluation(...)` must pass before publishing.

Published artifact:

- path: `global-patches/{batch_id}/global_patch.bin`
- bucket: `learning-deltas`
- checksum: SHA-256 over merged bytes

Database updates on success:

- mark participating `encrypted_delta_records.merge_batch_id = batch_id`
- finalize `merge_batches` with status `PUBLISHED`
- insert row into `global_patch_metadata`:
  - `version` (batch_id)
  - `checksum`
  - `published_at`
  - `download_url`

## 5) Registry and distribution to clients

Distribution endpoint:

- `GET /api/v1/federated/patch`

Source of truth:

- latest row in `global_patch_metadata`, ordered by `published_at DESC`.

Returned payload fields:

- `version`
- `checksum`
- `downloadUrl`
- `publishedAt`

This is the currently implemented client-facing delivery contract for global federated updates.

---

## Data model trace (Supabase)

Migration defines core tables:

- `merge_batches`
- `encrypted_delta_records`
- `global_patch_metadata`

All three are RLS-enabled with service role policy for backend writes.

Important observation for EVOS1-93:

- there is no separate first-class table for `global_user_adapter_metadata` vs `global_trainer_adapter_metadata` in the currently active schema.

---

## Parallel staging flow (non-canonical but active in repo)

Node staging subsystem (`api/staging`) provides a separate local-disk aggregation path:

1. `POST /staging/deltas` appends vectors into `staging_data/deltas/pool.json`.
2. `aggregateDeltas` sums vectors and writes `staging_data/aggregated/agg-*.json`.
3. `patcher.publishAggregatedAsPatch` gzips aggregate into `staging_data/updates/patch-*.json.gz` and emits checksum/signature metadata.
4. scheduler can run weekly and auto-publish.

This path demonstrates packaging/delivery mechanics, but does not represent distinct GU-vs-GT artifact families either.

---

## Where GU/GT appears vs where it executes

### Present in docs/spec narratives

Many docs describe intended GU/GT weekly training and distribution semantics.

### Not present as separate executable artifact paths

Current backend code paths implement:

- one encrypted delta ingest stream
- one merge pipeline
- one published global patch stream

No active code path currently branches publication into separate GU and GT artifacts.

---

## Responsibility split observed (device vs server)

Device/client responsibilities (as implied by ingest contract):

- produce local delta output
- encrypt payload
- send signed envelope with monotonic counter

Server responsibilities:

- authenticate/validate signed uploads
- persist encrypted deltas + metadata
- run batch merge and safety gating
- publish artifact metadata and download URL

For GU/GT specifically:

- server currently operates a **single global artifact lane**, not two distinct lanes.

---

## Direct answers for EVOS1-93

1. **Creation flow traced?** Yes — traced from encrypted client delta envelope through server merge output.
2. **Storage flow traced?** Yes — Supabase Storage `learning-deltas` + Supabase tables (`encrypted_delta_records`, `merge_batches`, `global_patch_metadata`).
3. **Distribution flow traced?** Yes — `/api/v1/federated/patch` returns latest metadata and download URL.
4. **Distinct GU and GT creation/storage/distribution currently implemented?** No.
   - Current implementation publishes a single global patch artifact channel.
5. **Canonical current path for governance docs?**
   - FastAPI federated path should be treated as canonical execution path.
   - Node staging path should be treated as staging/ops/testing auxiliary path.

---

## Recommended follow-ups (handoff to sibling issues)

- EVOS1-94: Decide whether canonical artifact model remains single global patch or moves to explicit GU + GT artifacts.
- EVOS1-91: Define shared federation pipeline contract with explicit artifact type field if GU/GT split is required.
- EVOS1-96: Formalize on-device vs server division for separate trainer/global-trainer channels if GT becomes first-class.
- EVOS1-95: Keep infrastructure migration (Fly replacement) scoped away from artifact semantics to avoid conflating concerns.