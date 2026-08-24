---
title: IMPLEMENTATION_PLAN
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/IMPLEMENTATION_PLAN.md
updated: 2026-07-24
---

# Diagnostics System Implementation Plan

## Overview

Add in-app diagnostics to debug model download/initialization failures on physical iOS devices without requiring Xcode logs.

## File Structure

### New Files to Create

1. **`flutter_app/lib/core/diagnostics/diag_log.dart`**
   - Singleton diagnostic logger with ring buffer
   - Disk persistence
   - URL/token redaction helpers

2. **`flutter_app/lib/core/diagnostics/diagnostics_overlay.dart`**
   - Full-screen diagnostics overlay widget
   - Stage indicator (NETWORK → DISK → VERIFY → INIT → READY)
   - Log viewer with timestamps
   - Export/share functionality
   - Reset cache and retry buttons

3. **`flutter_app/lib/core/diagnostics/device_info.dart`**
   - Device/environment info collection
   - iOS version, device model, storage, network type
   - Clock skew detection

4. **`flutter_app/lib/core/diagnostics/model_pipeline_stage.dart`**
   - Enum for pipeline stages
   - Stage transition helpers

### Files to Modify

1. **`flutter_app/lib/features/alice/domain/alice_asset_download_manager.dart`**
   - Add DiagLog instrumentation at NETWORK, DISK, VERIFY checkpoints
   - Add fail-fast error handling
   - Add HEAD request before download
   - Add streaming hash verification

2. **`flutter_app/lib/features/alice/presentation/ai_bootstrap_screen.dart`**
   - Add hidden gesture detector (7-tap on logo)
   - Integrate DiagnosticsOverlay
   - Show stage indicator in UI
   - Block progression unless stage == READY

3. **`flutter_app/lib/features/alice/presentation/alice_blob_avatar.dart`** (or main app entry)
   - Add long-press gesture detection as alternative trigger
   - Ensure DiagLog is initialized early

4. **`flutter_app/lib/ios/Runner/LlamaEngine.swift`** (future)
   - Add INIT stage logging (Swift-side)
   - Pass errors back to Flutter via method channel

## Implementation Order

1. **Phase 1: Core Logger** (DiagLog singleton)
2. **Phase 2: Overlay UI** (DiagnosticsOverlay widget)
3. **Phase 3: Instrumentation** (Add checkpoints to download manager)
4. **Phase 4: Integration** (Connect overlay to app, add gestures)
5. **Phase 5: Polish** (Error categorization, retry logic, export)

## Key Integration Points

- **Download start**: `AliceAssetDownloadManager.ensureAll()` → NETWORK stage
- **Download complete**: After file write → DISK stage
- **Verification**: After size/hash check → VERIFY stage
- **Model init**: `LlamaEngine.loadModel()` → INIT stage (requires Swift integration)
- **Ready**: After successful init → READY stage

## Related

^[source-materials/mirrors/doctrine/IMPLEMENTATION_PLAN.md]
