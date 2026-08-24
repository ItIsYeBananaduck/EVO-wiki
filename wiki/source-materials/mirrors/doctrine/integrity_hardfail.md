---
title: integrity_hardfail
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/integrity_hardfail.md"]
updated: 2026-07-24
---

# Integrity Hard-Fail Wiring Verification

**Date**: February 12, 2026
**Version**: v1.1.0-rc1

---

## Summary

The Alice asset download manager (`alice_asset_download_manager.dart`) implements three integrity verification layers for GGUF model files. All three produce **hard failures** — no silent fallback. When verification fails, the model does not load and the user sees an error screen.

---

## Hard-Fail Path 1: GGUF Magic Bytes Verification

**Code**: `_verifyGgufMagicBytes()` / `_verifyGgufMagicBytesFromFile()`

Reads the first 4 bytes of the file and checks for `0x47 0x47 0x55 0x46` ("GGUF").

**Failure behavior**:

- `verifyAll()` returns `false`
- `ai_bootstrap_screen.dart` throws `StateError('Downloaded files did not verify')`
- Error UI displayed: "Failed to download Alice assets"
- Model does NOT load

**Reproduction**:

1. Replace the first 4 bytes of the GGUF file with zeros
2. Launch app → bootstrap screen → verification fails → error shown
3. On next launch, `_ensureAsset()` detects invalid magic bytes, deletes file, re-downloads

**Telemetry**: Logged via `developer.log('AliceAssets: verification failed for ${asset.name}: invalid GGUF magic bytes')`

---

## Hard-Fail Path 2: File Size Mismatch

**Code**: `verifyAll()` and `_ensureAsset()` size checks

Two modes:

- **With `expectedSizeBytes`** (LoRA adapters): Exact match within 1% tolerance
- **Without `expectedSizeBytes`** (main model): Minimum 100 MB threshold

**Failure behavior**:

- `verifyAll()` returns `false` → `StateError` thrown → error UI
- `_ensureAsset()` deletes the corrupt file and re-downloads
- If re-download also fails verification, hard error persists

**Reproduction**:

1. Truncate the GGUF file to < 100 MB
2. Launch app → "file too small" logged → verification fails → error shown

**Telemetry**: Logged via `developer.log('AliceAssets: verification failed for ${asset.name}: size mismatch ...')`

---

## Hard-Fail Path 3: Download Write Verification

**Code**: `_ensureAsset()` post-write check (line ~970)

After downloading and writing bytes, immediately reads back the file length and compares to expected.

```dart
if (finalSize == null || finalSize != bytes.length) {
  throw StateError(
    'AliceAssets: File write failed! ${asset.name} expected ${bytes.length} bytes, got ${finalSize ?? "null"}',
  );
}
```

**Failure behavior**: `StateError` thrown immediately → caught by bootstrap screen → error UI

---

## Hard-Fail Path 4: R2 Download Size Mismatch

**Code**: `_downloadAssetStreamedToSharedStore()` / `_downloadAssetStreamedToFile()`

During streaming download, the `Content-Length` header from R2 is compared against `expectedSizeBytes`:

```
AliceAssets: SIZE MISMATCH - Expected: X bytes, R2 has: Y bytes
```

This is logged as a warning during download. The actual hard-fail occurs post-download via `verifyAll()`.

---

## Caller Chain (Hard-Fail Propagation)

```
AiBootstrapScreen._maybeStartDownloads()
  → assetManager.ensureAll()          // downloads all assets
  → assetManager.verifyAll()          // returns false if ANY check fails
  → if (!ok) throw StateError(...)    // HARD FAIL — no silent fallback
  → catch (e) { _error = '...' }     // error UI shown to user
```

In `nightly_model_sync.dart`:

```
NightlyModelSync.refreshAssets()
  → manager.ensureAll()
  → manager.verifyAll()
  → if (!ok) debugPrint('Asset verification failed')  // logged, nightly retry
```

---

## Verification Matrix

| Scenario                   | Detection                   | Action               | User Impact                            |
| -------------------------- | --------------------------- | -------------------- | -------------------------------------- |
| Bad GGUF magic bytes       | `_verifyGgufMagicBytes()`   | Delete + re-download | Error screen if re-download also fails |
| File too small (<100MB)    | Size check in `verifyAll()` | Delete + re-download | Error screen if re-download also fails |
| Size mismatch (adapter)    | 1% tolerance check          | Delete + re-download | Error screen if re-download also fails |
| Write failed (0 bytes)     | Post-write `StateError`     | Immediate throw      | Error screen                           |
| File not found             | `fileLength == null`        | Download from R2     | Normal flow                            |
| R2 Content-Length mismatch | Header comparison           | Warning log          | Post-download verify catches           |

---

## Key Observations

1. **No silent fallback**: Every verification failure either throws or returns `false`, which propagates to an error UI. The model never loads with corrupt data.

2. **Delete-and-retry**: `_ensureAsset()` deletes invalid files before re-downloading. This handles transient corruption (partial downloads, disk errors).

3. **Two verification points**: Both `_ensureAsset()` (pre-download check) and `verifyAll()` (post-download check) perform the same integrity checks, providing defense in depth.

4. **Android SAF limitation**: Magic bytes check is skipped for SAF storage (Android) because reading requires opening the entire file. Size verification still applies.

5. **Telemetry**: All failures are logged via `developer.log()` and `print()`. These appear in device console logs and can be captured via Xcode/Android Studio.

---

## Files Audited

- `flutter_app/lib/features/alice/domain/alice_asset_download_manager.dart` — all verification logic
- `flutter_app/lib/features/alice/presentation/ai_bootstrap_screen.dart` — hard-fail propagation
- `flutter_app/lib/core/background/nightly_model_sync.dart` — nightly retry on failure

## Related
