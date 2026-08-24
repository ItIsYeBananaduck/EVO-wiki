---
title: DIAGNOSTICS_IMPLEMENTATION_SUMMARY
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/DIAGNOSTICS_IMPLEMENTATION_SUMMARY.md
updated: 2026-07-24
---

# Diagnostics System - Implementation Summary

## ✅ Core Infrastructure Created

I've created the foundational diagnostic system with the following files:

### New Files Created

1. **`lib/core/diagnostics/diag_log.dart`** (245 lines)
   - Singleton `DiagLog` class with ring buffer (500 entries in memory)
   - Disk persistence (last 200 entries survive restarts)
   - URL/token redaction for security
   - Export functionality for diagnostics text

2. **`lib/core/diagnostics/diagnostics_overlay.dart`** (290 lines)
   - Full-screen diagnostics UI
   - Stage indicator with color coding
   - Scrollable log viewer with color-coded entries
   - Copy to clipboard functionality
   - Share diagnostics as text file
   - Reset cache and retry pipeline buttons

3. **`lib/core/diagnostics/model_pipeline_stage.dart`** (40 lines)
   - Enum for pipeline stages (idle, network, disk, verify, init, ready)
   - Display names and descriptions

4. **`lib/core/diagnostics/device_info.dart`** (75 lines)
   - Device/environment info collector
   - iOS version, device model, app version
   - Storage paths, locale, timezone

### Documentation Created

- **`IMPLEMENTATION_PLAN.md`** - High-level file structure and integration points
- **`DIAGNOSTICS_INSTRUMENTATION.md`** - Detailed code snippets for instrumentation
- **`DIAGNOSTICS_QUICK_START.md`** - Step-by-step implementation guide

### Dependencies Added

Added to `pubspec.yaml`:

- `device_info_plus: ^9.1.0`
- `package_info_plus: ^5.0.0`
- `share_plus: ^7.2.0`

## 📋 Next Steps to Complete Implementation

### 1. Install Dependencies (Required)

```bash
cd flutter_app
flutter pub get
```

### 2. Instrument Download Manager (Critical)

See `DIAGNOSTICS_INSTRUMENTATION.md` for complete code snippets. Key instrumentation points:

**NETWORK Stage:**

- Before download: Set stage, log start with asset info
- Perform HEAD request, log status/headers
- Detect error pages (text/html content-type)
- Log download progress and completion

**DISK Stage:**

- Before file write: Set stage, log target path
- Ensure directory exists
- After write: Verify file size (fail-fast if < 90% expected)
- Log file size and location

**VERIFY Stage:**

- Set stage to VERIFY
- Check GGUF magic bytes (0x47 0x47 0x55 0x46)
- Log verification result

### 3. Add Hidden Gesture (Required)

In `ai_bootstrap_screen.dart`, wrap the logo/avatar widget:

```dart
int _logoTapCount = 0;
DateTime? _lastTapTime;

Widget _buildLogoWithGesture(Widget logo) {
  return GestureDetector(
    onTap: () {
      final now = DateTime.now();
      if (_lastTapTime == null ||
          now.difference(_lastTapTime!) > const Duration(seconds: 2)) {
        _logoTapCount = 1;
      } else {
        _logoTapCount++;
      }
      _lastTapTime = now;

      if (_logoTapCount >= 7) {
        _logoTapCount = 0;
        Navigator.push(
          context,
          MaterialPageRoute(builder: (_) => const DiagnosticsOverlay()),
        ).then((result) {
          if (result == 'retry') {
            _init(); // Restart pipeline
          }
        });
      }
    },
    child: logo,
  );
}
```

### 4. Initialize DiagLog Early (Required)

In app startup (e.g., `main.dart` or app router initialization):

```dart
import 'package:flutter_app/core/diagnostics/diag_log.dart';

// After app initialization
DiagLog.instance.loadFromDisk();
```

### 5. Block Progression Until Ready (Recommended)

In `ai_bootstrap_screen.dart`, before allowing navigation:

```dart
import 'package:flutter_app/core/diagnostics/diag_log.dart';
import 'package:flutter_app/core/diagnostics/model_pipeline_stage.dart';

// Check before proceeding
if (DiagLog.instance.currentStage != ModelPipelineStage.ready) {
  // Show error UI or block navigation
  setState(() {
    _error = 'Model not ready. Check diagnostics for details.';
  });
  return;
}
```

## 🎯 Key Features Implemented

### ✅ Logging System

- Ring buffer (500 entries in memory)
- Disk persistence (200 entries)
- Automatic URL/token redaction
- Timestamped entries with levels (info/warn/error)

### ✅ Diagnostics Overlay

- Stage indicator with color coding
- Scrollable log viewer
- Error summary with expandable details
- Copy to clipboard
- Share as text file
- Reset cache button (needs implementation)
- Retry pipeline button

### ✅ Device Info Collection

- iOS version, device model
- App version/build
- Storage paths
- Locale/timezone
- Debug/release mode detection

## 🔄 What Still Needs Implementation

1. **Download Manager Instrumentation** - Add DiagLog calls at NETWORK/DISK/VERIFY checkpoints
2. **Hidden Gesture** - Add 7-tap gesture to bootstrap screen
3. **DiagLog Initialization** - Call `loadFromDisk()` at app startup
4. **Stage Blocking** - Block navigation until stage == READY
5. **Reset Cache Implementation** - Implement actual file deletion in download manager
6. **R2 Connectivity Test** - Add test endpoint and implementation (optional)
7. **Swift-side INIT Logging** - Add method channel calls from LlamaEngine (future)

## 📊 Usage Workflow

1. **Developer triggers diagnostics**: Tap logo 7 times
2. **View current stage**: Check banner at top
3. **Check errors**: Expand error summary if present
4. **Review logs**: Scroll through log entries
5. **Export diagnostics**: Copy or share for analysis
6. **Identify failing stage**: NETWORK → DISK → VERIFY → INIT
7. **Take action**: Reset cache or retry pipeline
8. **Fix issue**: Based on failing stage and error details

## 🔒 Security Features

- URLs redacted (query params stripped)
- Auth tokens/signatures redacted
- Presigned URLs never logged in full
- Only host + path logged for URLs

## 🎨 UI Features

- Color-coded stages (grey=idle, blue=in-progress, green=ready)
- Color-coded logs (red=error, orange=warn, black=info)
- Monospace font for logs
- Expandable error details
- Scrollable log viewer with auto-scroll to bottom
- Professional diagnostics export format

## 📝 Notes

- All logging is non-blocking (failures don't break app)
- Logs persist across app restarts
- Diagnostics export includes device info + all logs
- System is designed to be minimally invasive
- No changes to model architecture or inference logic

---

**Status**: Core infrastructure complete. Instrumentation and integration pending.

## Related

^[source-materials/mirrors/doctrine/DIAGNOSTICS_IMPLEMENTATION_SUMMARY.md]
