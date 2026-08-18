---
title: GGUF_FIX_SUMMARY
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/GGUF_FIX_SUMMARY.md"]
updated: 2026-07-24
---

# GGUF Download Fix Summary

## Problem

- Alice GGUF model not downloading reliably on iOS
- Size mismatches and Swift unable to find the file
- No clear logs showing worker URL, R2 key, or download status

## Solution

Implemented comprehensive logging and fixed path/size handling in `AliceAssetDownloadManager`.

## Changes Made

### Files Modified

1. `flutter_app/lib/features/alice/domain/alice_asset_download_manager.dart`
   - Added single-source logging before each asset download
   - Added worker configuration logging
   - Added token request/response logging
   - Added post-download size and magic bytes verification
   - Updated GGUF asset to use canonical `gguf/` prefix

2. `MODEL_DELIVERY.md` (new)
   - Documents canonical paths and workflow
   - Provides troubleshooting checklist
   - Includes evidence collection guide

### Key Fixes

- **Canonical R2 key**: Uses `models/alice-mistral-8b-q4.gguf` (R2 path: `alice-assets/models/alice-mistral-8b-q4.gguf`)
- **Worker logging**: Now logs resolved worker URL and environment variable
- **Token logging**: Confirms token request/response status
- **Download verification**: Logs Content-Length, streamed bytes, final size
- **Magic bytes verification**: Checks GGUF header immediately after download
- **Path alignment**: Ensures Swift and Dart use same local path

## Evidence Required

To verify the fix, collect these logs from Xcode console:

### 1. Worker Configuration

```
AliceAssets: worker config - env(ALICE_ASSET_WORKER_URL)=https://r2-importer.evoapp.workers.dev - resolved=https://r2-importer.evoapp.workers.dev
```

### 2. Asset Preparation

```
AliceAssets: ensuring models/alice-mistral-8b-q4.gguf worker=https://r2-importer.evoapp.workers.dev r2Key=alice-assets/models/alice-mistral-8b-q4.gguf target=AliceAssets/gguf/alice-mistral-8b-q4.gguf
```

### 3. Token Request

```
AliceAssets: download token fetched for alice-assets/models/alice-mistral-8b-q4.gguf (len=32)
```

### 4. Download Progress

```
AliceAssets: models/alice-mistral-8b-q4.gguf streamed 1073741824 of 4372815424 bytes
AliceAssets: gguf/alice-mistral-8b-q4.gguf streamed 2147483648 of 4372815424 bytes
AliceAssets: gguf/alice-mistral-8b-q4.gguf streamed 3221225472 of 4372815424 bytes
AliceAssets: gguf/alice-mistral-8b-q4.gguf streamed 4372815424 of 4372815424 bytes
```

### 5. Completion

```
AliceAssets: GGUF download completed for Alice Mistral 8B (GGUF) (models/alice-mistral-8b-q4.gguf) - finalSize=4372815424 bytes
AliceAssets: GGUF magic bytes check for Alice Mistral 8B (GGUF): OK
```

### 6. Swift Load

```
LlamaEngine: Model loaded successfully from /path/to/Documents/AliceAssets/gguf/alice-mistral-8b-q4.gguf
```

## Verification Steps

1. Build and run the iOS app
2. Trigger asset download (onboarding or manual trigger)
3. Monitor Xcode console for the above log sequence
4. Verify file exists at `Documents/AliceAssets/gguf/alice-mistral-8b-q4.gguf`
5. Confirm Swift LlamaEngine loads the model successfully

## Troubleshooting

If issues persist:

1. Check `ALICE_ASSET_WORKER_URL` environment variable
2. Verify worker allowlist includes `alice-assets/gguf/`
3. Confirm R2 object exists at correct key
4. Check Supabase authentication status
5. Verify SharedModelStore path resolution

## Impact

- **Reliability**: Clear logs make debugging much easier
- **Consistency**: Canonical paths ensure worker, client, and Swift align
- **Verification**: Magic bytes check ensures file integrity
- **Maintainability**: Documentation provides clear troubleshooting guide

## Related

^[{src_rel}]
