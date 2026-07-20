---
type: audit-finding
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-90 Audit: Active Fly.io Usage and Adapter Generation Paths

Date: 2026-03-31
Scope: codebase, configs, scripts, env references, and docs with focus on **active** adapter generation / aggregation / storage / distribution paths.

## Executive conclusion

- **Fly.io is not required by current active adapter-generation logic itself.**
- The active federated pipeline implementation is service-hosting agnostic in code and is primarily wired to **Supabase Storage + Supabase tables + Hugging Face distribution metadata**.
- **Fly.io is still present as deployment/runtime infrastructure configuration** (`federated-server/fly.toml`) and in operational docs/scripts, but there is no hard-coded Fly-only SDK dependency in the main merge/upload/publish code paths.
- A separate legacy/placeholder client module still references a placeholder Fly merge URL and appears non-canonical.

## Method

Ran repository-wide searches focused on Fly markers and adapter pipeline surfaces:

- `rg -n --hidden -S "fly\.io|flyctl|fly.toml|FLY_|Fly.io|Fly\.io|fly app|fly deploy|@fly|\bfly\b"`
- `rg -n "VITE_FLY_EXERCISES_URL|VITE_FLY_FOOD_URL|exercisesUrl|foodsUrl|FLY_MERGE_URL|your-fly-endpoint|flyctl|fly\.dev|federated-server|/merge|adaptive-exercises|adaptive-foods" app src api scripts supabase federated-server README.md README_FEDERATED_LEARNING.md .env.example`
- Focused code inspection in `federated-server/src`, `api/staging`, `app/src/lib/config.ts`, and `src/lib/model.ts`.

## Findings by pipeline stage

### 1) Adapter/delta ingestion (active)

- Canonical federated upload endpoint exists at `POST /api/v1/federated/upload-delta` in `federated-server/src/api/upload_delta.py`.
- Uploaded encrypted deltas are written to Supabase Storage bucket `learning-deltas` and metadata is recorded in Supabase (`encrypted_delta_records` via RPC/table writes).
- No Fly-specific code dependency is required for this logic; Fly appears only as a potential hosting target.

### 2) Aggregation/merge generation (active)

- `merge_deltas` in `federated-server/src/ml/gguf_merger.py` drives active merge behavior:
  - Reads pending rows from `encrypted_delta_records`.
  - Downloads encrypted blobs from Supabase Storage.
  - Decrypts + averages deltas.
  - Writes merged artifact to Supabase Storage (`global-patches/.../global_patch.bin`).
  - Publishes metadata to `global_patch_metadata`.
- Merge orchestration is invoked from `federated-server/src/api/aggregate.py`.
- Again, no hard Fly runtime API usage in merge code path.

### 3) Artifact metadata + distribution (active)

- `GET /api/v1/federated/patch` in `federated-server/src/api/publish_model.py` serves latest patch metadata from `global_patch_metadata`.
- Distribution endpoint returns metadata (version/checksum/downloadUrl/publishedAt); source-of-truth is Supabase state.
- Main API composition in `federated-server/src/api/main.py` includes upload, aggregate, canary, and publish routers without Fly SDK coupling.

### 4) Staging/approval path (active but non-Fly merge execution)

- `api/staging/index.js` contains approval comments mentioning Fly merge/HF upload but explicitly states actual merge/upload are out of scope in this handler (manual approval only).
- This indicates Fly wording exists in comments/ops context, while execution for that route does not actually call Fly.

## Fly.io-specific surfaces discovered

### Infrastructure/config (still present)

- `federated-server/fly.toml` defines a Fly deployment target for the federated server.
- Shell/batch deployment scripts and docs contain Fly CLI instructions and `*.fly.dev` references.

### Environment/config references (partially active or dormant)

- `.env.example` includes `VITE_FLY_EXERCISES_URL` and `VITE_FLY_FOOD_URL`.
- `app/src/lib/config.ts` exposes these as `API_CONFIG.exercisesUrl` and `API_CONFIG.foodsUrl`.
- Search results show no active call sites consuming those config keys in the app runtime path, suggesting dormant or incomplete feature wiring.

### Legacy/placeholder code

- `src/lib/model.ts` defines `FLY_MERGE_URL = 'https://your-fly-endpoint.example.com/merge'` and posts deltas there.
- URL is a placeholder and this module is outside the federated-server canonical backend path; treat as legacy/non-canonical unless explicitly wired by runtime entrypoints.

## Determination: Is Fly part of active adapter generation/aggregation/storage/distribution?

- **Adapter generation/aggregation/storage/distribution logic:** **No hard Fly dependency** found in active backend code path.
- **Deployment/runtime hosting choice:** **Yes, Fly remains configured** as a deployment option/target for federated-server.
- **Legacy and documentation surface:** extensive historical Fly references remain and can obscure current canonical flow.

## Recommended follow-up for EVOS1 parent stream

1. **Canonicalize one pipeline document** (source-of-truth) that names exact endpoints, data stores, and artifact tables.
2. **Mark dormant Fly env vars and legacy placeholder module** (`src/lib/model.ts`) as deprecated or remove if unused.
3. **Separate “historical/spec” Fly references from active operations docs** to reduce governance/audit ambiguity.
4. If migration off Fly is intended, treat this issue as evidence that replacement is mostly an **infrastructure/runtime decision**, not a deep code rewrite in merge logic.
