---
type: audit-finding
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-94 Audit: Canonical Global Adapter Artifact Model

Date: 2026-03-31  
Scope: Determine whether the codebase currently implements a real canonical **global adapter artifact model** (including GU/GT split semantics), or whether GU/GT remain conceptual/partial/inconsistent.  
References:

- `docs/audits/EVOS1-93-gu-gt-creation-storage-distribution-flow.md`
- `docs/audits/EVOS1-90-flyio-active-path-audit.md`

## Executive decision

- A **canonical implemented artifact lane does exist** today, but it is a **single global patch artifact** lane, not a typed global adapter model.
- There is **no first-class canonical GU/GT artifact model** in active schema, API contracts, or merge publication logic.
- Therefore, GU/GT are still **conceptual/planned** in architecture docs, while production-like executable flow remains a single artifact class: `global_patch`.

## What is canonical in code today

The currently executable and cross-validated path has one artifact identity and one distribution contract:

1. Aggregation publishes exactly one artifact path pattern:
   - `learning-deltas/global-patches/{batch_id}/global_patch.bin`
2. Publication metadata is persisted in exactly one table:
   - `global_patch_metadata`
3. Client fetch contract exposes one payload shape through one endpoint:
   - `GET /api/v1/federated/patch` returning `version/checksum/downloadUrl/publishedAt`

This is stable enough to call the **canonical implemented artifact**, but that artifact is not GU- or GT-typed.

## Evidence: artifact model is single-lane (not GU/GT-typed)

### Schema and storage evidence

- Supabase migration defines `global_patch_metadata` as the only global federated artifact metadata table.
- No corresponding `global_user_adapter_metadata`, `global_trainer_adapter_metadata`, or generic `artifact_type` discriminator exists in the active migration set.
- Merge process writes a single `artifact_path` into `merge_batches` and inserts one row into `global_patch_metadata`.

### API contract evidence

- `federated-server/src/api/publish_model.py` documents and implements global patch distribution with `global_patch_metadata` as source of truth.
- Returned response model (`FederatedPatchMetadata`) has no adapter family discriminator (`GU`, `GT`, or equivalent).

### Merge implementation evidence

- `merge_deltas` computes one averaged vector and uploads one binary (`global_patch.bin`) per successful batch.
- It then inserts one metadata row in `global_patch_metadata` without any type dimension.

### Test evidence

- Federated patch tests validate reads from `global_patch_metadata` and 404 behavior when empty.
- Merge tests assert one `global_patch_metadata` insert on publish.

## GU/GT status classification

## 1) GU/GT in documentation: **Present**

GU/GT appear heavily in planning/specification docs and architecture narratives.

## 2) GU/GT in executable backend artifact model: **Absent as first-class model**

No active runtime path creates separate GU and GT publication channels with independent metadata, versioning, or distribution endpoints.

## 3) GU/GT in canonical contract surface: **Not codified**

No enum/discriminator in schema/API identifies artifact family; no API endpoint offers per-family retrieval (e.g., GU latest vs GT latest).

## Final determination for EVOS1-94

- **Does a canonical global adapter artifact model exist?**
  - **Yes, but only as a single global patch artifact model.**
- **Does a canonical GU/GT split model exist in implementation?**
  - **No. GU/GT remain conceptual/partial/inconsistent relative to executable code paths.**

## Decision required (governance)

Choose one explicit target model and formalize it in EVOS1-91 pipeline definition:

### Option A — Officially canonize single-lane model (near-term)

- Treat `global_patch` as the sole global artifact type.
- Update docs/spec language to stop implying active GU/GT split.
- Keep schema/API as-is, possibly adding explicit naming (`artifact_kind = global_patch`) only for clarity.

### Option B — Implement typed global adapter model (GU/GT)

- Add first-class artifact typing in schema (`artifact_type` or separate GU/GT tables).
- Split publication metadata and retrieval contracts by type.
- Introduce deterministic merge/publication jobs per artifact family.
- Update client/runtime to consume typed global artifacts.

## Recommended immediate next step

For EVOS1 governance consistency, adopt Option A as current truth in docs now, and open a scoped implementation plan if Option B is desired. This avoids treating planned GU/GT semantics as already implemented.