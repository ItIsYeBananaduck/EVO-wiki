---
title: CACHE_ISSUE_FIX
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CACHE_ISSUE_FIX.md"]
updated: 2026-07-24
---

# Cache Issue Fix for Adapter Downloads

## Problem

Even after deleting and re-uploading fixed adapters to R2, the app keeps downloading the old `phi3` architecture versions.

## Root Cause

The app downloads adapters through a Cloudflare Worker (`r2-importer.evoapp.workers.dev`) which:

1. Sets `Cache-Control: private, max-age=3600` (1 hour cache)
2. Cloudflare's edge cache may also cache files
3. Even after R2 files are updated, cached versions may be served

## Solution

### 1. Updated Worker Code (✅ Done)

Modified `r2-worker/src/index.ts` to disable caching for adapters:

- Adapters now return `Cache-Control: no-cache, no-store, must-revalidate`
- This ensures fresh downloads after updates

### 2. Deploy Updated Worker

**You need to manually deploy the updated worker:**

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Navigate to **Workers & Pages** → **r2-importer**
3. Copy the updated code from `r2-worker/src/index.ts`
4. Paste into the worker editor
5. Click **Save and Deploy**

### 3. Purge Cloudflare Cache (Optional but Recommended)

After deploying the worker, purge the cache:

1. In Cloudflare Dashboard → **Workers & Pages** → **r2-importer**
2. Go to **Settings** → **Triggers**
3. Look for cache purge options, OR
4. Use Cloudflare API to purge cache for adapter paths:
   ```bash
   curl -X POST "https://api.cloudflare.com/client/v4/zones/{zone_id}/purge_cache" \
     -H "Authorization: Bearer {api_token}" \
     -H "Content-Type: application/json" \
     --data '{"files":["https://r2-importer.evoapp.workers.dev/download?key=adapters/enforcer/enforcer_lora.gguf","https://r2-importer.evoapp.workers.dev/download?key=adapters/voice/voice_lora.gguf"]}'
   ```

### 4. Verify R2 Files Are Correct

Already verified:

- ✅ ENF adapter in R2 has `llama` architecture
- ✅ VOICE adapter in R2 has `llama` architecture
- ✅ Files were deleted and re-uploaded

### 5. Clear App Cache

Already done:

- ✅ Deleted cached adapters in simulator
- ✅ App will re-download on next launch

## After Deployment

Once the worker is deployed with the no-cache headers:

1. Restart the app
2. It will download fresh adapters (no cache)
3. Adapters should have correct `llama` architecture

## Alternative: Add Version Query Parameter

If cache issues persist, we can modify the Flutter app to add a version/timestamp query parameter to bust the cache:

```dart
final Uri uri = Uri.parse('$_workerBaseUrl/download?key=${Uri.encodeComponent(r2Key)}&v=${DateTime.now().millisecondsSinceEpoch}');
```

But this should not be necessary once the worker is deployed with no-cache headers.

## Related

^[{src_rel}]
