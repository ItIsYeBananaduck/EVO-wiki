---
title: Model Storage Architecture — R2
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Model Storage Architecture — R2.md
updated: 2026-07-24
---

# Model Storage Architecture — R2

> NOTE: This is a canonical doctrine note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

---

## Purpose

Define where AI model artifacts are stored, how clients download them, and what security contracts govern the delivery flow. Cloudflare R2 is the canonical artifact store; clients never download directly from HuggingFace or Supabase Storage.

---

## Core Principle

Clients download model artifacts **only** from Cloudflare R2 via the R2 Worker delivery flow, using short-lived worker-issued tokens. Checksum and size must be verified before model activation.

---

## Definitions

- **R2 Worker** — `r2-importer.evoapp.workers.dev`, the authenticated delivery proxy in front of Cloudflare R2
- **Download token** — short-lived token issued by the R2 Worker; scoped to one object key; TTL 60–300 seconds
- **Model metadata** — server-side record containing `r2_object_key`, `sha256_checksum`, `size_bytes`, `version`

---

## System Structure

### Canonical Model Paths (current)

| Model | Format | R2 path | Used by |
|---|---|---|---|
| alice-human-fusion | GGUF Q4_K_M | `models/alice-human-fusion.Q4_K_M.gguf` | `LlamaEngine.swift`, Android llama.cpp |

> Note: MLX safetensors paths (`alice-assets/models/alice-phi3-mlx-base-q4/`) are deprecated. The current stack uses llama.cpp with GGUF on all platforms.

### Download Flow

1. Client fetches latest model metadata from backend control plane (version + object key + checksums)
2. Client requests short-lived download token:
   ```http
   POST https://r2-importer.evoapp.workers.dev/download/token
   Authorization: Bearer <supabase_access_token>
   Body: { "key": "<r2_object_key>", "ttlSeconds": 300 }
   ```
3. Client downloads artifact (`<r2_object_key>` must be URL-encoded / percent-encoded):
   ```http
   GET https://r2-importer.evoapp.workers.dev/download?key=<url_encoded_r2_object_key>
   Authorization: Bearer <download_token>
   ```
4. Client verifies `sha256_checksum` and `size_bytes` before install/activation

### Model Metadata Contract

```json
{
  "version": "2026.04.20",
  "r2_object_key": "models/alice-human-fusion.Q4_K_M.gguf",
  "sha256_checksum": "<hex_sha256>",
  "size_bytes": 4294967296
}
```

---

## Rules

- **Never bundle HuggingFace tokens** in client builds
- **Never construct client download URLs** to HuggingFace or Supabase Storage buckets
- **Always use worker-issued download tokens** for artifact bytes
- **Always verify checksum and size** before model activation
- **Treat `r2_object_key` as opaque** — clients must not mutate path conventions
- Object location rotation must be handled by metadata updates, not hard-coded client paths
- Keep download token TTL short: 60–300 seconds

---

## Flow

Client entry point is `flutter_app/lib/features/alice/domain/alice_asset_download_manager.dart`. The manager calls the backend for metadata, requests a download token, streams bytes from the R2 Worker, verifies integrity, then hands off to the platform model loader (`LlamaEngine.swift` on iOS, llama.cpp JNI on Android).

---

## Relationships

See also: [[EVO On-Device First Principle]], [[EVOLoRA Mesh]], [[LoRA Artifact Sync]]

---

## Edge Cases / Special Handling

- Deprecated model paths in R2: `alice-mistral-8b-q4.gguf`, `alice-3b-q4.gguf`, `alice-3b-q8.gguf`, `alice-phi3-mlx-base-q4/` — these should be removed via `scripts/cleanup-old-r2-models.sh`
- When adding a new model: update metadata backend, then update this note's canonical path table

---

## Summary

AI model artifacts are delivered exclusively via the R2 Worker with short-lived download tokens. Clients obtain metadata from the backend, request a token, stream bytes, verify integrity, then activate. HuggingFace and Supabase Storage are not client download origins. Current canonical format is GGUF Q4_K_M loaded by llama.cpp.

## Related

^[source-materials/mirrors/doctrine/Model Storage Architecture — R2.md]
