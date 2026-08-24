---
title: GET_LOGS_VIA_APPCONNECT
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/GET_LOGS_VIA_APPCONNECT.md
updated: 2026-07-24
---

# Getting Crash Logs via AppConnect

Since the app freezes and method channels don't work, use these methods to access logs via AppConnect:

## Method 1: Access Log Files Directly (Easiest)

The logs are automatically written to files in the app's Documents directory:

### Files Created:

1. **`crash_logs.txt`** - Full detailed log file
2. **`CRASH_LOG_SUMMARY.txt`** - Summary file (updated every 10 seconds)

### Location:

```
/var/mobile/Containers/Data/Application/[APP_ID]/Documents/crash_logs.txt
/var/mobile/Containers/Data/Application/[APP_ID]/Documents/CRASH_LOG_SUMMARY.txt
```

### Via AppConnect:

1. **Open AppConnect** on your cloud Mac
2. **Navigate to device file system**
3. **Go to:** `/var/mobile/Containers/Data/Application/[APP_ID]/Documents/`
4. **Download these files:**
   - `crash_logs.txt` (full logs)
   - `CRASH_LOG_SUMMARY.txt` (summary with recent entries)

The summary file is updated every 10 seconds and contains:

- Recent crashes
- Recent errors
- Recent log entries
- Statistics

## Method 2: Check Summary File First

The `CRASH_LOG_SUMMARY.txt` file is easier to read and contains:

- Last 10 crashes
- Last 20 errors
- Last 50 log entries
- Statistics

This is updated automatically, so even if the app freezes, the last update will show where it stopped.

## Method 3: Export Logs to Shareable Location

Add this to your Flutter app to copy logs to a shareable location:

```dart
// Copy logs to a file that can be shared
Future<void> exportLogsToShareable() async {
  const platform = MethodChannel('evo/crash_logger');
  final logs = await platform.invokeMethod('getLogContents');

  // Save to a file in a shareable location
  // This depends on your file sharing setup
}
```

## Method 4: Automatic Export on Freeze

The logs are written synchronously to the file, so even if the app freezes:

1. **All logs up to the freeze point are saved**
2. **The file is flushed after each log entry**
3. **You can access it via AppConnect even if app is frozen**

## What to Look For

In the logs, search for:

- `Step 1:` through `Step 7:` - Shows initialization progress
- `TIMED OUT` - Metal warmup timeout
- `INVARIANT_VIOLATION` - Pre-crash validation failures
- `[FATAL]` or `[ERROR]` - Critical errors
- Last log entry - Shows where it froze

## Quick Check

1. **Open AppConnect**
2. **Navigate to Documents folder**
3. **Open `CRASH_LOG_SUMMARY.txt`**
4. **Check the "Last Updated" timestamp** - if it's recent, the app was running
5. **Check "RECENT LOG ENTRIES"** - the last entry shows where it stopped
6. **Check "RECENT CRASHES"** - see if any crashes were detected

The summary file is your best bet since it's automatically updated and contains the most important information.

## Related

^[source-materials/mirrors/doctrine/GET_LOGS_VIA_APPCONNECT.md]
