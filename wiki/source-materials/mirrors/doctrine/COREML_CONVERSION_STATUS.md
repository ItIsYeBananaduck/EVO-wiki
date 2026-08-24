---
title: COREML_CONVERSION_STATUS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/COREML_CONVERSION_STATUS.md"]
updated: 2026-07-24
---

# CoreML Conversion Status

## ❌ Direct ONNX → CoreML Conversion Failed

**Issue**: Dynamic shapes in Kokoro ONNX model

```
Error: Converter is not implemented (OperationDescription(domain='', operation_type='Shape', version=19))
```

**Root Cause**: Kokoro uses dynamic sequence lengths which `onnx2torch` doesn't support for direct conversion.

## ✅ Code Changes Complete

The iOS code is **already updated** to:

- Prefer CoreML when available
- Fallback to ONNX if CoreML fails
- This avoids ONNX conflicts with llama.cpp

**Current Status**:

- Code: ✅ Ready for CoreML
- Model: ⏳ Needs proper CoreML conversion

## 🔄 Next Steps: Proper CoreML Conversion

The `kokoro-coreml` project uses a **two-stage pipeline**:

1. **Duration model** (fixed shapes)
2. **Decoder model** (fixed-shape buckets, e.g. 3s)

This requires:

- Exporting Kokoro from PyTorch source (not ONNX)
- Using fixed-shape variants
- More complex but production-ready

**Repository**: https://github.com/mattmireles/kokoro-coreml

## 🧪 Testing Plan

### Option 1: Test Current Setup (Recommended)

Test with ONNX first - our code changes should prevent conflicts:

- CoreML code is in place
- ONNX fallback works
- No runtime conflicts (ONNX only loads if CoreML unavailable)

### Option 2: Full CoreML Conversion

Use kokoro-coreml pipeline for proper conversion (requires more setup)

## 📝 Current Workflow

**For Now:**

1. Test iOS with current ONNX model (should work with our fixes)
2. Verify no ONNX/llama.cpp conflicts
3. Schedule CoreML conversion using kokoro-coreml approach

**Later:**

1. Set up kokoro-coreml conversion pipeline
2. Export fixed-shape CoreML models
3. Upload to R2
4. Update asset manager

## Related
