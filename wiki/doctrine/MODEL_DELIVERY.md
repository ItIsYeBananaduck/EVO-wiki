---
title: MODEL_DELIVERY
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/MODEL_DELIVERY.md
updated: 2026-07-24
---

# Model Delivery Documentation

## Overview

This document describes the end-to-end delivery workflow for Alice models (GGUF, ONNX, voices) from R2 to iOS/Android devices.

## Architecture

- **Source**: Supabase Storage bucket `alice-assets`
- **Worker**: Cloudflare Worker that issues short-lived download tokens
- **Client**: Flutter app using `AliceAssetDownloadManager`
- **Storage**: SharedModelStore (iOS file system or Android SAF)

## Canonical Paths

### Remote R2 Keys

- GGUF: `alice-assets/models/alice-mistral-8b-q4.gguf`
- ONNX models: `alice-assets/onnx/<filename>.onnx`
- Tokenizer: `alice-assets/onnx/tokenizer.json`
- Voices: `alice-assets/onnx/voice_<name>.bin`

### Local Paths (iOS)

- Documents directory: `~/Documents/AliceAssets/gguf/alice-mistral-8b-q4.gguf`
- AppGroup (if configured): `~/AppGroup/group.biz.lsctech.adaptivefit/AliceAssets/gguf/alice-mistral-8b-q4.gguf`

### Local Paths (Android)

- SAF storage: User-selected directory/EVO/ModelStore/AliceAssets/gguf/alice-mistral-8b-q4.gguf

## Worker Configuration

### Environment Variable

- `ALICE_ASSET_WORKER_URL` (default: `https://r2-importer.evoapp.workers.dev`)
- Set in `.env` for iOS builds

### Endpoints

- `POST {worker}/download/token` - Request download token
- `GET {worker}/download?key=<r2Key>` - Download file with token

### Allowlist

Worker must allow these prefixes:

- `alice-assets/models/`
- `alice-assets/onnx/`
- `alice-assets/models/` (legacy)

## Download Flow

1. **Token Request**

   ```dart
   POST {worker}/download/token
   Authorization: Bearer <supabase access token>
   Body: {"key": "alice-assets/models/alice-mistral-8b-q4.gguf"}
   Response: {"token": "..."}
   ```

2. **File Download**

   ```dart
   GET {worker}/download?key=alice-assets/models/alice-mistral-8b-q4.gguf
   Authorization: Bearer <download token>
   Headers: Range: bytes=0- (for resume)
   Response: 200 (fresh) or 206 (resume)
   ```

3. **Verification**
   - Size check: Must match `expectedSizeBytes` exactly for GGUF
   - Magic bytes: First 4 bytes must be "GGUF" (0x47 0x47 0x55 0x46)
   - Location: Must be at `Documents/AliceAssets/gguf/alice-mistral-8b-q4.gguf`

## Asset Configuration

### GGUF Asset

```dart
_AliceAsset(
  name: 'Alice Mistral 8B (GGUF)',
  storagePath: 'models/alice-mistral-8b-q4.gguf',
  relativeTarget: 'AliceAssets/gguf/alice-mistral-8b-q4.gguf',
  useStreaming: true,
  expectedSizeBytes: 4372815424, // 4.073 GB
)
```

## Verification Checklist

### Pre-Download

- [ ] Worker URL logged: `AliceAssets: worker config - env(ALICE_ASSET_WORKER_URL)=... - resolved=...`
- [ ] R2 key logged: `AliceAssets: ensuring models/alice-mistral-8b-q4.gguf worker=... r2Key=alice-assets/models/alice-mistral-8b-q4.gguf target=AliceAssets/gguf/alice-mistral-8b-q4.gguf`

### Token Request

- [ ] Token request status: `200`
- [ ] Token response: `{"token": "..."}`

### Download

- [ ] Download status: `200` (fresh) or `206` (resume)
- [ ] Content-Length header matches expected size
- [ ] Bytes streamed reach expected size
- [ ] Final file size: `4372815424` bytes

### Post-Download

- [ ] File exists at correct path
- [ ] GGUF magic bytes: `OK`
- [ ] Swift loader finds file: `Model loaded successfully`

## Troubleshooting

### Size Mismatch

- Check R2 object size via worker logs
- Update `expectedSizeBytes` in `_assets` list
- Delete partial file before retry

### Token Errors

- Verify Supabase session is valid
- Check worker allowlist includes key prefix
- Ensure worker URL is correct

### Swift Can't Find File

- Verify SharedModelStore path resolution
- Check AppGroup configuration on iOS
- Ensure download completed successfully

## Evidence Collection

To verify the fix, collect these logs from Xcode console:

1. **Worker Configuration**

   ```
   AliceAssets: worker config - env(ALICE_ASSET_WORKER_URL)=https://r2-importer.evoapp.workers.dev - resolved=https://r2-importer.evoapp.workers.dev
   ```

2. **Token Request**

   ```
   AliceAssets: download token fetched for alice-assets/models/alice-mistral-8b-q4.gguf (len=32)
   ```

3. **Download Progress**

   ```
   AliceAssets: models/alice-mistral-8b-q4.gguf streamed 1073741824 of 4372815424 bytes
   ```

4. **Completion**

   ```
   AliceAssets: GGUF download completed for Alice Mistral 8B (GGUF) (models/alice-mistral-8b-q4.gguf) - finalSize=4372815424 bytes
   AliceAssets: GGUF magic bytes check for Alice Mistral 8B (GGUF): OK
   ```

5. **Swift Load**
   ```
   LlamaEngine: Model loaded successfully from /path/to/Documents/AliceAssets/gguf/alice-mistral-8b-q4.gguf
   ```

## Maintenance

- Update `expectedSizeBytes` when R2 object changes
- Verify worker allowlist matches asset paths
- Test both fresh download and resume scenarios
- Periodically verify magic bytes check works for new GGUF versions

## Related

^[source-materials/mirrors/doctrine/MODEL_DELIVERY.md]
