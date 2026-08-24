---
title: CAPTURE_SYSTEM_CRASH_LOGS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CAPTURE_SYSTEM_CRASH_LOGS.md"]
updated: 2026-07-24
---

# Capturing System Crash Logs

If crashes aren't appearing in our custom logs, they may be captured by iOS's system crash reporting. Here's how to access them:

## Method 1: Via Xcode (if you have direct access)

1. **Open Xcode**
2. **Window → Devices and Simulators** (Cmd+Shift+2)
3. **Select your device**
4. **Click "View Device Logs"**
5. **Look for your app's crash reports**

## Method 2: Via AppConnect (Cloud Mac)

System crash logs are stored in:

```
/var/mobile/Library/Logs/CrashReporter/
```

### Steps:

1. **Open AppConnect**
2. **Navigate to:** `/var/mobile/Library/Logs/CrashReporter/`
3. **Look for files named:** `YourAppName_YYYY-MM-DD-HHMMSS_DeviceName.crash`
4. **Download the most recent crash file**

## Method 3: Via Console.app (if available)

1. **Open Console.app** on your Mac
2. **Connect device via USB**
3. **Select your device** in the sidebar
4. **Filter by your app name**
5. **Look for crash reports**

## Method 4: Check TestFlight Crash Reports

If you're using TestFlight:

1. **Open App Store Connect**
2. **Go to TestFlight → Crashes**
3. **Select your app**
4. **Download crash reports**

## What to Look For

In system crash logs, look for:

- **Exception Type:** SIGABRT, SIGSEGV, etc.
- **Crashed Thread:** Which thread crashed
- **Stack Trace:** Shows where in the code it crashed
- **Binary Images:** Shows loaded libraries

## Our Custom Logs vs System Logs

- **Custom logs (`crash_logs.txt`)**: Capture our own logging and signal handlers
- **System logs**: Capture crashes that bypass our handlers or happen at a lower level

**If crashes aren't in custom logs, they're likely in system logs.**

## Quick Check Script

You can add this to your app to list recent system crash logs:

```swift
func listSystemCrashLogs() {
    let crashDir = "/var/mobile/Library/Logs/CrashReporter/"
    let fileManager = FileManager.default

    if let contents = try? fileManager.contentsOfDirectory(atPath: crashDir) {
        let crashFiles = contents.filter { $0.contains("YourAppName") }
        print("Found \(crashFiles.count) crash logs")
        for file in crashFiles.sorted().suffix(5) {
            print("  - \(file)")
        }
    }
}
```

## Important Notes

- System crash logs require device access (not available via method channel if app is frozen)
- Crash logs are only created if the app actually crashes (not for freezes)
- If the app freezes without crashing, you'll only see logs up to the freeze point in our custom logs

## Related
