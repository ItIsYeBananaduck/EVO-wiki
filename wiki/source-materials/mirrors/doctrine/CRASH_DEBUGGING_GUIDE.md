---
title: CRASH_DEBUGGING_GUIDE
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CRASH_DEBUGGING_GUIDE.md"]
updated: 2026-07-24
---

# Crash Debugging Guide

If the app is crashing with no visible logs, use these methods to find the crash:

## 1. Check Xcode Console

The crash logs are printed to console immediately. In Xcode:

- Open **View → Debug Area → Activate Console** (or Cmd+Shift+Y)
- Look for lines starting with `[CrashLogger]` or `[LlamaEngine]`
- Look for `========== CRASH DETECTED ==========` or `INVARIANT_VIOLATION`

## 2. Retrieve Logs via Flutter

Add this to your Flutter code to retrieve crash logs:

```dart
import 'package:flutter/services.dart';

Future<void> checkCrashLogs() async {
  try {
    const platform = MethodChannel('evo/crash_logger');

    // Get log file path
    final logPath = await platform.invokeMethod('getLogFileURL');
    print('Log file: $logPath');

    // Get recent logs (last 100 lines)
    final recentLogs = await platform.invokeMethod('getRecentLogs', {'count': 100});
    print('Recent logs:\n$recentLogs');

    // Get full log contents
    final fullLogs = await platform.invokeMethod('getLogContents');
    print('Full logs:\n$fullLogs');

    // Get log file info
    final logInfo = await platform.invokeMethod('getLogFileInfo');
    print('Log info: $logInfo');
  } catch (e) {
    print('Error retrieving logs: $e');
  }
}
```

## 3. Check Device Logs via Xcode

1. Connect device to Mac
2. Open **Window → Devices and Simulators**
3. Select your device
4. Click **View Device Logs**
5. Look for crashes with your app's bundle ID
6. Check the crash report for stack traces

## 4. Check Crash Log File Location

The crash log file is stored at:

```
/var/mobile/Containers/Data/Application/[APP_ID]/Documents/crash_logs.txt
```

To access it:

- Use Xcode's **Window → Devices and Simulators → Download Container**
- Or use the Flutter method channel above to retrieve contents

## 5. Enable More Verbose Logging

All critical operations now log to:

- **Console** (immediate, synchronous)
- **Crash log file** (persistent, async)

Look for these log prefixes:

- `[FATAL]` - Critical errors that could cause crashes
- `[ERROR]` - Errors that might cause issues
- `[WARN]` - Warnings about potential problems
- `[INFO]` - General information
- `[DEBUG]` - Detailed debugging info

## 6. Common Crash Points

The following operations now have extensive logging:

1. **safeDecode** - Every decode operation logs entry/exit
2. **loadModel** - Model loading with Metal warmup
3. **\_processGeneration** - Full request processing
4. **resizeContextIfNeeded** - Context resizing operations
5. **Invariant violations** - All validation failures

## 7. If Still No Logs

If you see no logs at all, the crash might be happening:

- Before CrashLogger initialization (very early in app lifecycle)
- In a background thread without logging
- In a system framework (not our code)

In this case:

1. Check Xcode's crash reports (Window → Devices → View Device Logs)
2. Enable **Edit Scheme → Run → Diagnostics → Enable Address Sanitizer**
3. Enable **Edit Scheme → Run → Diagnostics → Enable Thread Sanitizer**
4. Check if crash happens in simulator vs device (Metal issues)

## 8. Quick Test

Add this to your Flutter app startup to verify logging works:

```dart
@override
void initState() {
  super.initState();
  WidgetsBinding.instance.addPostFrameCallback((_) async {
    const platform = MethodChannel('evo/crash_logger');
    await platform.invokeMethod('getLogFileInfo');
    print('Crash logger initialized');
  });
}
```

## Related
