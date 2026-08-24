---
title: HOW_TO_ACCESS_DEVICE_FILES
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/HOW_TO_ACCESS_DEVICE_FILES.md
updated: 2026-07-24
---

# How to Access Device Files (for Crash Logs)

Since you're on a cloud Mac and can't connect directly, here are your options:

## Option 1: Xcode Devices Window (if Xcode is available)

1. **Open Xcode** on your cloud Mac
2. **Window → Devices and Simulators** (Cmd+Shift+2)
3. **Connect your device via USB** (or it should appear if already connected)
4. **Select your device** in the left sidebar
5. **Select your app** in the "Installed Apps" section
6. **Click "Download Container..."** button
7. **Extract the downloaded .xcappdata file**
8. **Navigate to:** `AppData/Documents/`
9. **Find:**
   - `crash_logs.txt`
   - `CRASH_LOG_SUMMARY.txt`

## Option 2: Apple Configurator 2 (if installed)

1. **Open Apple Configurator 2** (download from Mac App Store if needed)
2. **Connect device via USB**
3. **Select your device**
4. **Right-click your app** → **Add/Remove Apps** → **Download App Data**
5. **Extract and navigate to Documents folder**

## Option 3: Terminal/Command Line (if device is connected)

If you have terminal access and the device is connected:

```bash
# List connected devices
xcrun devicectl list devices

# If device is connected, you can use idevice tools
# (requires libimobiledevice to be installed)
idevice_id -l  # List device UDIDs
```

## Option 4: Flutter Method Channel (if app is running)

If the app is still running (even if frozen), you can try to get logs via Flutter:

```dart
// In your Flutter code
final platform = MethodChannel('evo/crash_logger');
final logs = await platform.invokeMethod('getRecentLogs', {'count': 200});
print(logs);
```

## Option 5: TestFlight Crash Reports

If you're using TestFlight:

1. **Go to App Store Connect** (https://appstoreconnect.apple.com)
2. **TestFlight → Your App → Crashes**
3. **Download crash reports** (these are system crash logs, not our custom logs)

## Option 6: Xcode Console (if device is connected)

1. **Open Xcode**
2. **Window → Devices and Simulators**
3. **Select your device**
4. **Click "Open Console"** button
5. **Filter by your app name**
6. **Look for crash logs**

## Option 7: Share Extension (if you add one)

You could add a share extension to your app that exports logs:

```swift
// In your app, add a button to share logs
@IBAction func shareLogs() {
    let logPath = CrashLogger.shared.getLogFileURL()
    if let logData = try? Data(contentsOf: URL(fileURLWithPath: logPath)) {
        let activityVC = UIActivityViewController(activityItems: [logData], applicationActivities: nil)
        present(activityVC, animated: true)
    }
}
```

## What You're Looking For

The crash logs are in:

```
/var/mobile/Containers/Data/Application/[APP_ID]/Documents/crash_logs.txt
/var/mobile/Containers/Data/Application/[APP_ID]/Documents/CRASH_LOG_SUMMARY.txt
```

## Quick Check: Is Device Connected?

Run this in terminal:

```bash
xcrun devicectl list devices
```

If you see your device listed, you can access it via Xcode.

## If You Can't Access Device Files Directly

**Alternative approach:** Add a debug menu in your app that:

1. Shows recent logs on screen
2. Allows copying logs to clipboard
3. Allows sharing logs via email/airdrop

Would you like me to add a debug UI for viewing logs directly in the app?

## Related

^[source-materials/mirrors/doctrine/HOW_TO_ACCESS_DEVICE_FILES.md]
