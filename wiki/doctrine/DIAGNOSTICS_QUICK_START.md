---
title: DIAGNOSTICS_QUICK_START
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/DIAGNOSTICS_QUICK_START.md
updated: 2026-07-24
---

# Diagnostics System - Quick Start Guide

## Summary

A comprehensive in-app diagnostics system for debugging model download/initialization failures on physical iOS devices without requiring Xcode logs.

## Files Created

1. **`lib/core/diagnostics/diag_log.dart`** - Singleton logger with ring buffer and disk persistence
2. **`lib/core/diagnostics/diagnostics_overlay.dart`** - Full-screen diagnostics UI
3. **`lib/core/diagnostics/model_pipeline_stage.dart`** - Pipeline stage enum
4. **`lib/core/diagnostics/device_info.dart`** - Device/environment info collector

## Files to Modify

1. **`pubspec.yaml`** - Add dependencies (device_info_plus, package_info_plus, share_plus)
2. **`lib/features/alice/domain/alice_asset_download_manager.dart`** - Add instrumentation
3. **`lib/features/alice/presentation/ai_bootstrap_screen.dart`** - Add hidden gesture and overlay integration

## Implementation Steps

### Step 1: Install Dependencies

```bash
cd flutter_app
flutter pub get
```

### Step 2: Add Instrumentation to Download Manager

See `DIAGNOSTICS_INSTRUMENTATION.md` for detailed code snippets. Key points:

- Add DiagLog imports at top of file
- Set stage to NETWORK before download starts
- Perform HEAD request and log status/headers
- Set stage to DISK before file write
- Verify file size after write (fail-fast if < 90% of expected)
- Set stage to VERIFY and check magic bytes for GGUF files
- Log all errors with context

### Step 3: Add Hidden Gesture Trigger

In `ai_bootstrap_screen.dart`:

- Wrap logo/avatar widget with GestureDetector
- Track tap count (reset after 2 seconds of no taps)
- Open DiagnosticsOverlay after 7 taps
- Handle retry callback to restart pipeline

### Step 4: Initialize DiagLog Early

In app startup (e.g., main.dart or app router):

```dart
// Initialize and load persisted logs
DiagLog.instance.loadFromDisk();
```

### Step 5: Block Progression Until Ready

In `ai_bootstrap_screen.dart`, check stage before proceeding:

```dart
if (DiagLog.instance.currentStage != ModelPipelineStage.ready) {
  // Show error or block navigation
  return;
}
```

## Usage

### Opening Diagnostics

- **Primary method**: Tap the app logo/avatar 7 times quickly (within 2 seconds)
- **Alternative**: Long-press corner of screen (if implemented)

### Using Diagnostics

1. **View Current Stage**: Top banner shows current pipeline stage
2. **View Error Summary**: Expandable error details if any errors occurred
3. **View Logs**: Scrollable log viewer with color-coded entries (ERROR=red, WARN=orange)
4. **Copy Diagnostics**: Copies all logs + device info to clipboard
5. **Share Diagnostics**: Exports to text file and opens iOS share sheet
6. **Reset Cache**: Deletes downloaded model files (requires implementation)
7. **Retry Pipeline**: Restarts from NETWORK stage

## Key Features

### Fail-Fast Rules

- **Network**: Stop if status != 2xx, or content-type is text/html (error page)
- **Disk**: Stop if file size < 90% of expected, or write fails
- **Verify**: Stop if magic bytes mismatch or hash verification fails

### Error Categorization

- **401/403**: Auth/access denied → Regenerate presigned URL
- **404**: Model not found → Check configuration
- **5xx/408/429**: Temporary server issue → Retry with backoff
- **Disk errors**: Storage/permission issues → Check free space
- **Integrity errors**: Corrupted file → Delete and retry

### Security

- URLs are redacted (query params stripped)
- Auth tokens/signatures are redacted
- Full presigned URLs never logged

## Next Steps

1. Run `flutter pub get` to install dependencies
2. Implement instrumentation in download manager (see DIAGNOSTICS_INSTRUMENTATION.md)
3. Add hidden gesture to bootstrap screen
4. Test on physical device
5. Use diagnostics to identify failing stage
6. Implement targeted fixes based on diagnostics output

## Troubleshooting

### Diagnostics overlay doesn't open

- Check that gesture detector is properly attached to logo widget
- Verify tap count is resetting correctly (should reset after 2s of no taps)

### Logs are empty

- Ensure DiagLog.instance.log() is being called in download manager
- Check that loadFromDisk() is called on app startup
- Verify SharedPreferences is working

### Export/share doesn't work

- Check share_plus plugin is properly installed
- Verify file permissions on iOS
- Check that path_provider is returning correct directory

## Related

^[source-materials/mirrors/doctrine/DIAGNOSTICS_QUICK_START.md]
