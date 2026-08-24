---
title: Gap_To_100_Percent_Compliance_Proposal
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Gap_To_100_Percent_Compliance_Proposal.md
updated: 2026-07-24
---

# Gap To 100% Compliance Proposal (EVOLoRA Mesh v1 + EVOFL)

This document is an **ordered implementation plan** to close all known gaps found during the audit.

**Rules used**

- Only items with explicit code evidence are treated as implemented.
- Anything implicit/assumed is treated as **non-compliant**.

---

## Phase 0 — Make federated learning pipeline actually functional (highest leverage)

### 0.1 Unify client↔server upload contract (blocking)

- **Problem**: Capacitor client upload payload does not match server schema.
  - Client sends:
    - `app/src/lib/services/ml/deltaQueue.ts` `uploadDeltaEntry()` body includes `delta_id`, `training_steps`, `checksum`, `encrypted_payload`, `salt`, `instance_key`.
  - Server expects:
    - `federated-server/src/api/upload_delta.py` `LearningDeltaUpload`: `user_id`, `model_version`, `delta_weights`, `validation_metrics`, `size_bytes`.
- **Fix** (choose one, but do it end-to-end):
  - **Option A (recommended)**: Update server to accept client contract.
    - Add a new Pydantic model aligned to `deltaQueue.ts` and keep `/deltas/upload` for legacy.
    - Ensure route exists at `/api/v1/federated/upload-delta` to match `modelUpdater.ts` patch route style.
  - **Option B**: Update client to match server `LearningDeltaUpload`.
    - Requires generating `validation_metrics` client-side (currently `deltaQueue.ts` does not provide it).
- **Acceptance test**:
  - A device can enqueue, encrypt, and successfully upload a delta; server writes an `encrypted_delta_records` row and stores blob in Supabase Storage.

### 0.2 Fix route prefix mismatch (blocking)

- **Problem**: Server routes are under `/api/v1` via `federated-server/src/api/main.py` `include_router(..., prefix="/api/v1")`, but client posts to `${FEDERATED_API_URL}/federated/upload-delta`.
- **Fix**:
  - Either:
    - Change client to POST to `${FEDERATED_API_URL}/api/v1/federated/upload-delta` in `app/src/lib/services/ml/deltaQueue.ts`.
    - Or mount an alias route at root (not recommended).

### 0.3 Enforce upload authentication + signature verification (blocking security)

- **Problem**:
  - Upload endpoints are not protected by `verify_api_key` (`federated-server/src/api/main.py` only protects aggregation/canary).
  - Client sends `X-Federated-Signature` but server does not verify it (`federated-server/src/api/upload_delta.py`).
- **Fix**:
  - Implement signature verification in server upload handler.
    - Client signing: `app/src/lib/services/ml/deltaQueue.ts` `signPayload(JSON.stringify(body), hmacKey)`.
    - Key derivation currently uses `PBKDF2(password=modelVersion, salt=metadata.salt)` in `deltaQueue.ts` `deriveKeys()`.
  - Decide authoritative key scheme (see Security plan): move to device-bound secret.
- **Acceptance test**:
  - Server rejects missing/invalid signature.

---

## Phase 1 — Make EVOFL crypto real (device-bound, replay-safe)

### 1.1 Device-bound key material for delta encryption (blocking privacy)

- **Problem**: `app/src/lib/services/privacy/deltaEncryptor.ts` keeps `encryptionKey` in-memory and generates it; no OS keystore binding is shown.
- **Fix**:
  - Store per-device secret in Keychain/Keystore (Capacitor secure storage plugin) and derive keys from it.
  - Rotate key on reinstall; ensure old queued deltas are invalidated or re-encrypted.

### 1.2 Replay protection + monotonic counters

- **Problem**: Client includes `delta_id` and signature, but no explicit anti-replay checks exist on server.
- **Fix**:
  - Server stores `delta_id` uniqueness + a monotonic counter tied to `instance_key` (client field in `deltaQueue.ts`).
  - Reject duplicates and out-of-window timestamps.

---

## Phase 2 — Complete the federated merge pipeline (non-stub)

### 2.1 Implement real merge (decrypt → validate → average)

- **Problem**: `federated-server/src/ml/gguf_merger.py` has TODO; it only assigns `merge_batch_id` and marks batch as PUBLISHED.
- **Fix**:
  - Implement:
    - Download encrypted delta payloads from Supabase Storage (bucket used by server: `learning-deltas` in `federated-server/src/api/upload_delta.py`).
    - Decrypt (requires coherent key scheme; currently referenced as `decrypt_delta` in `gguf_merger.py`).
    - Anomaly detection and federated averaging.
    - Write resulting global patch artifact + record metadata.

### 2.2 Replace stub safety evaluation

- **Problem**: `federated-server/src/ml/safety_evaluator.py` always returns `passed: True`.
- **Fix**:
  - Implement an actual evaluation suite and block publish if failed.

### 2.3 Publish patch metadata to a stable source of truth

- **Problem**:
  - Supabase has `global_patch_metadata` table (`20251207123800_create_federated_delta_tables.sql`) but patch endpoint reads `get_latest_model_version` RPC (`federated-server/src/api/publish_model.py`).
- **Fix**:
  - Decide whether “global patch” is a GGUF model version or a separate artifact.
  - If separate, update `/api/v1/federated/patch` to read from `global_patch_metadata`.

---

## Phase 3 — EVOLoRA Mesh v1 core (routing/gating/weights)

### 3.1 Implement explicit adapter types + router

- **Problem**: No explicit EVOLoRA mesh router/gating rules exist in codebase.
- **Fix**:
  - Add a central “adapter routing” module with explicit adapter inventory and selection rules.
  - Ensure plan generation uses router output; ensure inference applies multiple adapters with documented weighting.

### 3.2 Immutable plan versioning & approval pipeline

- **Problem**: Local plan storage lacks immutable versioning.
  - Evidence: `flutter_app/lib/features/alice/domain/plans_store.dart` stores `TrainingPlanRecord` without explicit version hashes/immutability.
- **Fix**:
  - Create plan version objects with content hash; store append-only history.
  - Require trainer approval for certain actions (ties into existing queue function `supabase/functions/queue-trainer-approval/index.ts`).

---

## Phase 4 — Weekly reporting + divergence capture

### 4.1 Divergence capture in workout logging

- **Problem**: Workout logging exists but “divergence vs plan” capture is not proven end-to-end in this audit set.
- **Fix**:
  - Ensure session logs include planned vs actual and divergence reasons.

### 4.2 Weekly encrypted report envelope

- **Problem**: No explicit weekly encrypted report artifact is proven end-to-end.
- **Fix**:
  - Generate weekly summary payload, encrypt (reuse chat crypto patterns), sign, upload, and enforce retention.

---

## Phase 5 — Hardening + compliance closure

### 5.1 Replace JS timers with OS-level background execution

- **Problem**: `backgroundModelUpdater.ts` uses `setTimeout` and can miss windows.
- **Fix**:
  - Use native background task APIs for iOS/Android (Capacitor background task plugin / WorkManager / BGTaskScheduler).

### 5.2 Add regression tests for contract + security

- **Must include**:
  - Upload route contract tests.
  - Signature verification tests.
  - Replay prevention tests.
  - Merge pipeline tests with known deltas.

---

## Completion criteria (definition of “100% compliant”)

- Device produces encrypted deltas with device-bound keys.
- Server authenticates and verifies signatures + anti-replay.
- Merge actually decrypts/aggregates and produces a published global patch with real safety gating.
- EVOLoRA mesh router exists and is the authoritative path for plan/inference.
- Plan versioning is immutable and approvals are enforced.
- Weekly reporting is encrypted, signed, and retention-controlled.
- Background scheduling uses OS-level jobs and is measurable/auditable.

## Related

^[source-materials/mirrors/doctrine/Gap_To_100_Percent_Compliance_Proposal.md]
