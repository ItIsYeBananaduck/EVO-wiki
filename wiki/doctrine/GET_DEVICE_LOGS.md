---
title: GET_DEVICE_LOGS
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/GET_DEVICE_LOGS.md
updated: 2026-07-24
---

# How to Get Crash Logs from Physical Device

## Method 1: Xcode Device Logs (Easiest)

1. **Connect device to Mac via USB**
2. **Open Xcode** → Window → Devices and Simulators (Cmd+Shift+2)
3. **Select your device** from the left sidebar
4. **Click "Open Console"** button (or View Device Logs)
5. **Filter by your app name** or look for recent crashes
6. **Look for entries with:**
   - `[CrashLogger]`
   - `[LlamaEngine]`
   - `INVARIANT_VIOLATION`
   - `CRASH DETECTED`
   - `SIGABRT` or `SIGSEGV`

## Method 2: Download App Container (Get Log File)

1. **Connect device to Mac**
2. **Xcode** → Window → Devices and Simulators
3. **Select your device**
4. **Select your app** under "Installed Apps"
5. **Click "Download Container..."**
6. **Extract the .xcappdata file** (right-click → Show Package Contents)
7. **Navigate to:** `AppData/Documents/crash_logs.txt`
8. **Open the file** to see all crash logs

## Method 3: Flutter Method Channel (If App Still Runs)

If the app can still launch (even briefly), add this to your Flutter code:

```dart
import 'package:flutter/services.dart';

Future<void> getCrashLogs() async {
  try {
    const platform = MethodChannel('evo/crash_logger');

    // Get log file info
    final info = await platform.invokeMethod('getLogFileInfo');
    print('Log file info: $info');

    // Get recent logs
    final recentLogs = await platform.invokeMethod('getRecentLogs', {'count': 500});
    print('=== RECENT CRASH LOGS ===');
    print(recentLogs);

    // Get full logs
    final fullLogs = await platform.invokeMethod('getLogContents');
    print('=== FULL CRASH LOGS ===');
    print(fullLogs);
  } catch (e) {
    print('Error: $e');
  }
}
```

Call this immediately on app startup or from a debug button.

## Method 4: Xcode Console During Run

1. **Connect device**
2. **Run app from Xcode** (Cmd+R)
3. **Open Console** (Cmd+Shift+Y)
4. **Watch for logs in real-time**
5. **When crash happens**, logs will be visible in console

## Method 5: sysdiagnose (System Logs)

If the app crashes completely, system logs may have info:

1. **On device:** Settings → Privacy & Security → Analytics & Improvements → Analytics Data
2. **Look for:** `Runner-*.ips` files (crash reports)
3. **Share to Mac** or view on device

## Quick Test Script

Add this to your Flutter app to test logging:

```dart
// In your main.dart or a debug screen
void testCrashLogger() async {
  const platform = MethodChannel('evo/crash_logger');

  // Test logging
  await platform.invokeMethod('getLogFileInfo');

  // Get logs
  final logs = await platform.invokeMethod('getRecentLogs', {'count': 50});
  debugPrint('=== CRASH LOGS ===');
  debugPrint(logs);
}
```

## What to Look For

In the logs, search for:

- `FATAL` - Critical errors
- `ERROR` - Errors that may cause crashes
- `INVARIANT_VIOLATION` - Pre-crash validation failures
- `safeDecode` - Decode operations
- `loadModel` - Model loading
- `Metal` - Metal-related issues
- `SIGABRT` - Abort signal crashes
- `SIGSEGV` - Segmentation fault

## If Still No Logs

If you see NO logs at all, the crash might be:

1. **Before CrashLogger initializes** - Check AppDelegate startup
2. **In a system framework** - Check Xcode crash reports
3. **Memory issue** - Check device memory warnings
4. **Metal driver crash** - Check system logs

Enable Address Sanitizer:

- Edit Scheme → Run → Diagnostics → Enable Address Sanitizer
- This will catch memory issues and show detailed stack traces

## Related

^[source-materials/mirrors/doctrine/GET_DEVICE_LOGS.md]
