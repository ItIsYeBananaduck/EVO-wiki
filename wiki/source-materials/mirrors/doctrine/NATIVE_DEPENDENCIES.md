---
title: NATIVE_DEPENDENCIES
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/NATIVE_DEPENDENCIES.md"]
updated: 2026-07-24
---

# Unavoidable Native Dependencies (EVOTRA-478)

## Overview

This document lists all native (Swift/iOS) dependencies that **must remain in native code** even after the Dart talent migration (EVOTRA-463). These are architectural constraints, not technical debt.

## Model Inference (LlamaEngine.swift)

**Status**: REQUIRED — Cannot move to Dart

The on-device LLM inference runs through llama.cpp, which is a C/C++ library with Swift bindings. Dart cannot directly call llama.cpp.

### Native Responsibilities:
- Tokenization (Qwen2.5 151K vocabulary)
- Context management (n_ctx = 4096, n_batch = 512)
- KV cache management (n_seq_max = 2 for talents)
- Sampling chain (DRY, min_p, temperature)
- Token generation loop

### Dart Interface:
```dart
// MethodChannel calls for inference
_channel.invokeMethod('generate', {
  'prompt': prompt,
  'maxTokens': maxTokens,
  'temperature': temperature,
});
```

### Why Not Dart:
- llama.cpp is C/C++ with no Dart bindings
- Metal GPU acceleration requires native code
- Memory-mapped model loading (GGUF) is platform-specific

## OCR / Vision APIs

**Status**: REQUIRED — Cannot move to Dart

Apple's Vision framework provides on-device OCR, which is significantly better than any Dart package.

### Native Responsibilities:
- `VNRecognizeTextRequest` for nutrition labels
- `VNRecognizeTextRequest` for body composition scans
- PDF rendering for Styku/DEXA/InBody reports

### Dart Interface:
```dart
// MethodChannel calls for OCR
_channel.invokeMethod('scanText', {'imageData': bytes});
```

### Why Not Dart:
- Apple's Vision framework is iOS-native only
- Significantly more accurate than Tesseract/OCR Dart packages
- On-device ML model (no network required)

## TTS (Supertonic/SupertonicTtsPlugin.swift)

**Status**: REQUIRED — Cannot move to Dart

The Supertonic TTS system uses ONNX Runtime with custom Kokoro voice models.

### Native Responsibilities:
- ONNX model loading and inference
- Audio synthesis pipeline
- Audio playback coordination

### Dart Interface:
- EventChannel for streaming audio chunks
- MethodChannel for voice selection/speed

### Why Not Dart:
- ONNX Runtime has no Dart bindings
- Audio synthesis requires low-latency native code

## Keychain / Secure Storage

**Status**: REQUIRED — Cannot move to Dart

iOS Keychain provides hardware-backed encryption that Dart packages cannot replicate.

### Native Responsibilities:
- Encryption key storage (`ed_priv.pem`, `ed_pub.pem`)
- Secure Enclave operations (if available)

### Dart Interface:
```dart
// MethodChannel calls for key access
_channel.invokeMethod('getSigningKey');
```

### Why Not Dart:
- Hardware security requires native Keychain access
- Flutter's `flutter_secure_storage` is a thin wrapper around native anyway

## Some Tool Executions

**Status**: PARTIAL — Some can move, some cannot

### Tools That Stay Native:
- `scan_nutrition_label` (depends on Vision OCR)
- `scan_body_composition` (depends on Vision OCR)

### Tools That Are Already Dart:
- `update_profile` → Dart `UserFitnessProfileStore`
- `update_nutrition_targets` → Dart `NutritionTargetService`
- `save_custom_food` → Dart `CustomFoodStore`
- `plan.create` → Dart `PlansStore`

## Networking / WebSockets

**Status**: ALREADY DART

The networking layer is already in Dart using `dart:io` WebSocket implementations. No native dependencies here.

## Platform-Specific Features

### Background Processing
- BGProcessingTask for nightly training (native)
- Hive/Swarm discovery (native + Dart coordination)

### Notifications
- Local notifications (native)
- Push notifications (would be native if implemented)

### Widgets
- iOS 16+ Lock Screen widgets (native SwiftUI)
- Home Screen widgets (native SwiftUI)

## Summary Table

| Dependency | Native | Dart | Reason |
|------------|--------|------|--------|
| LLM Inference | ✓ | ✗ | llama.cpp C/C++ |
| OCR/Vision | ✓ | ✗ | Apple Vision framework |
| TTS/Supertonic | ✓ | ✗ | ONNX Runtime |
| Secure Storage | ✓ | ✗ | iOS Keychain |
| Network/WebSocket | ✗ | ✓ | `dart:io` |
| Profile/Plan Stores | ✗ | ✓ | Dart SQLite |
| Background Tasks | ✓ | ✗ | iOS BGProcessingTask |
| Widgets | ✓ | ✗ | iOS SwiftUI |

## Future Possibilities

### Could Move to Dart (with significant effort):
1. **Some tools** — Already happening via ActionRuntime executors
2. **Simple inference** — If a pure-Dart ONNX runtime emerges

### Will Always Stay Native:
1. **llama.cpp inference** — Too complex to reimplement
2. **Apple Vision OCR** — Platform-specific ML models
3. **Hardware security** — Requires native Keychain

## Migration Impact

The Dart talent migration (EVOTRA-463) does not eliminate these native dependencies. Instead, it:

- **Before**: Native owned talent routing + step orchestration + inference
- **After**: Dart owns routing + orchestration, native only for inference

This is a **separation of concerns**, not a full native elimination.

## Related Files

### Native (Must Keep):
- `ios/Runner/LlamaEngine.swift`
- `ios/Runner/SupertonicTtsPlugin.swift`
- `ios/Runner/BodyScanVisionHandler.swift`
- `ios/Runner/NutritionScanVisionHandler.swift`

### Dart (New Source of Truth):
- `lib/features/alice/domain/talents/` — All talent definitions
- `lib/features/alice/domain/action_runtime/` — Action execution

## Acceptance Criteria (EVOTRA-478)

- [x] Document all native dependencies
- [x] Explain why each cannot move to Dart
- [x] List Dart interface boundaries
- [x] Create summary table
- [x] Note future possibilities
