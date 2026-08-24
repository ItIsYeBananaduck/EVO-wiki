---
title: MODEL_DOWNLOAD_FLOW
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/MODEL_DOWNLOAD_FLOW.md"]
updated: 2026-07-24
---

# Model Download Flow (R2-Only)

## Policy

Client applications must download AI/TTS model artifacts **only** from Cloudflare R2 via the R2 Worker delivery flow. HuggingFace and Supabase Storage are not valid client download origins.

## End-to-End Client Flow

1. Client fetches latest model metadata from backend control plane (for example, latest version endpoint/RPC).
2. Backend metadata includes the R2 object key and integrity fields:
   - `r2_object_key`
   - `sha256_checksum`
   - `size_bytes`
3. Client requests a short-lived download token from the R2 Worker:
   - `POST https://r2-importer.evoapp.workers.dev/download/token`
   - Header: `Authorization: Bearer <supabase_access_token>`
   - Body: `{ "key": "<r2_object_key>", "ttlSeconds": 300 }` (`ttlSeconds` optional)
   - Response: `{ "download_token": "..." }`
4. Client downloads the model bytes from the R2 Worker:
   - `GET https://r2-importer.evoapp.workers.dev/download?key=<r2_object_key>`
   - Header: `Authorization: Bearer <download_token>`
   - Response: streamed model bytes from R2
5. Client verifies downloaded bytes against `sha256_checksum` and `size_bytes` before install/activation.

## API Contract (Client-Side Expectations)

### Model metadata contract

```json
{
  "version": "2026.04.20",
  "r2_object_key": "models/alice-human-fusion.Q4_K_M.gguf",
  "sha256_checksum": "<hex_sha256>",
  "size_bytes": 4294967296
}
```

### Download token request

```http
POST /download/token HTTP/1.1
Host: r2-importer.evoapp.workers.dev
Authorization: Bearer <supabase_access_token>
Content-Type: application/json

{
  "key": "models/alice-human-fusion.Q4_K_M.gguf",
  "ttlSeconds": 300
}
```

```json
{
  "download_token": "<short_lived_download_token>"
}
```

### Artifact download request

```http
GET /download?key=models/alice-human-fusion.Q4_K_M.gguf HTTP/1.1
Host: r2-importer.evoapp.workers.dev
Authorization: Bearer <download_token>
```

Response: binary stream of the model object from Cloudflare R2.

## Reference Client Pseudocode

```typescript
async function downloadLatestModel(): Promise<void> {
  const latest = await getLatestModelMetadata();
  // expected fields: r2_object_key, sha256_checksum, size_bytes

  const tokenRes = await fetch(
    "https://r2-importer.evoapp.workers.dev/download/token",
    {
      method: "POST",
      headers: {
        Authorization: `Bearer ${supabaseAccessToken}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        key: latest.r2_object_key,
        ttlSeconds: 300,
      }),
    },
  );

  const { download_token } = await tokenRes.json();

  const bytesRes = await fetch(
    `https://r2-importer.evoapp.workers.dev/download?key=${encodeURIComponent(latest.r2_object_key)}`,
    {
      headers: {
        Authorization: `Bearer ${download_token}`,
      },
    },
  );

  const bytes = new Uint8Array(await bytesRes.arrayBuffer());
  await verifySha256AndSize(bytes, latest.sha256_checksum, latest.size_bytes);
  await installModel(bytes, latest.version);
}
```

## Security Requirements

- Do not bundle HuggingFace tokens in client builds.
- Do not construct client download URLs to HuggingFace or Supabase Storage buckets.
- Always require short-lived worker-issued download tokens for artifact bytes.
- Always verify checksum and size before model activation.

## Operational Notes

- Keep token TTL short (for example, 60-300 seconds).
- Treat `r2_object_key` as opaque; clients should not mutate path conventions.
- Rotation or migration of object locations must be handled by metadata updates, not hard-coded client paths.

---

Related notes: [[R2_MODEL_PATHS]], [[model-id-client]], [[integrity_hardfail]], [[update_applier]]

## Related
