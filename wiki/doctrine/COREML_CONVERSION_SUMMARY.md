---
title: COREML_CONVERSION_SUMMARY
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/COREML_CONVERSION_SUMMARY.md
updated: 2026-07-24
---

# Kokoro TTS CoreML Conversion - Summary

## ✅ Changes Completed

### Code Updated to Prefer CoreML

The `KokoroTtsPlugin.swift` has been updated to **prefer CoreML over ONNX** on real devices to avoid runtime conflicts.

**Key Changes:**

1. CoreML is now checked first before ONNX
2. ONNX is only used as a fallback if CoreML is unavailable or fails
3. This eliminates ONNX runtime conflicts with llama.cpp

**Execution Order:**

```
1. Try CoreML (preferred - no ONNX conflicts)
   ↓ (if fails)
2. Try ONNX (fallback)
   ↓ (if fails)
3. Use System TTS (final fallback)
```

## 🔄 Complete Workflow: Convert & Upload

### Step 1: Convert ONNX → CoreML

```bash
# Convert using the existing script
python3 scripts/convert_kokoro_to_coreml.py \
  <path_to_onnx/kokoro-v1-fp16.onnx> \
  <output/kokoro-v1.mlpackage>

# The script outputs:
# - kokoro-v1.mlpackage (CoreML format)
```

**Note**: You can download the ONNX model from R2 first if needed:

- R2 Key: `alice-assets/onnx/kokoro-v1-fp16.onnx`
- Or find it in app container: `/EVO/ModelStore/AliceAssets/onnx/`

### Step 2: Upload CoreML to R2

```bash
# Upload to R2 using the upload script
python3 upload_to_r2.py \
  <path_to/kokoro-v1.mlpackage> \
  "alice-assets/onnx/kokoro-v1.mlpackage"

# R2 Configuration:
# - Bucket: evostorage
# - Key: alice-assets/onnx/kokoro-v1.mlpackage
# - Credentials: From flutter_app/.env (CF_R2_ACCESS_KEY_ID, etc.)
```

### Step 3: Update Asset Manager

Update `flutter_app/lib/features/alice/domain/alice_asset_download_manager.dart` to include CoreML model:

```dart
_AliceAsset(
  name: 'Kokoro TTS v1 CoreML',
  storagePath: 'onnx/kokoro-v1.mlpackage',  // NEW
  relativeTarget: 'AliceAssets/onnx/kokoro-v1.mlpackage',
  useStreaming: true,
  expectedSizeBytes: <size_in_bytes>,  // Update after conversion
),
```

### Step 4: On-Device Compilation

The app can compile `.mlpackage` to `.mlmodelc` on first use:

- CoreML automatically compiles when loading
- Or pre-compile: `xcrun coremlc compile kokoro-v1.mlpackage output_dir/`

### Model Storage Paths

**R2 Storage:**

- Key: `alice-assets/onnx/kokoro-v1.mlpackage`

**Device Storage (after download):**

- iOS: `EVO/ModelStore/AliceAssets/onnx/kokoro-v1.mlpackage` (or `.mlmodelc` if compiled)
- Android: N/A (CoreML is iOS-only)

## 🎯 Benefits

1. **No ONNX Runtime Conflicts** - CoreML uses native Apple frameworks
2. **Compatible with llama.cpp** - Both use Metal/native APIs
3. **Works on Simulator** - CoreML works on simulator (ONNX had issues)
4. **Better Performance** - Can use Apple Neural Engine (ANE)
5. **Future-Proof** - CoreML is Apple's recommended on-device ML framework

## 📝 Notes

- The code will automatically try CoreML first, then fallback to ONNX if needed
- Once CoreML model is available, it will be used by default
- ONNX code remains as fallback for compatibility
- No breaking changes - existing ONNX setup still works

## Related

^[source-materials/mirrors/doctrine/COREML_CONVERSION_SUMMARY.md]
