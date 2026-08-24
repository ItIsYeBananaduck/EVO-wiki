---
title: BACKGROUND_MODEL_UPDATES
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/BACKGROUND_MODEL_UPDATES.md"]
updated: 2026-07-24
---

# Background Model Updates

## Overview

The app automatically checks for and downloads alice-human-fusion model updates at night (3 AM) while respecting battery life, network conditions, and user preferences.

## How It Works

### 🌙 Nighttime Schedule

**Time:** 3:00 AM daily
**Reason:** User is likely sleeping, phone is likely charging and on WiFi

```
3:00 AM - Wake up background task
3:00 AM - Check battery (> 20%)
3:00 AM - Check for model update
3:00 AM - Detect network type (WiFi/cellular)
3:00 AM - Auto-download (WiFi) or Prompt (cellular)
3:15 AM - Task complete, go back to sleep
```

### 📡 Network Detection

**WiFi Detected:**

```
✅ Auto-download model
✅ Silent notification when complete
✅ Model ready for next workout
```

**Cellular Detected:**

```
🔔 Send notification: "New model available (~4GB)"
📱 User can tap to approve or decline
⏰ If declined, retry tomorrow at 3 AM
```

**No Connection:**

```
⏭️  Skip download
🔄 Retry tomorrow at 3 AM
```

### 🔋 Battery Protection

**Before Download:**

- Check battery level
- If < 20% (configurable): Skip download
- If battery saver mode: Skip download
- Retry tomorrow

**Result:** Never drains battery unexpectedly

## User Settings

Users can configure via Settings → Model Updates:

| Setting                       | Options | Default |
| ----------------------------- | ------- | ------- |
| **Automatic Night Updates**   | On/Off  | On      |
| **Auto-Download on WiFi**     | On/Off  | On      |
| **Auto-Download on Cellular** | On/Off  | Off ⚠️  |
| **Minimum Battery**           | 10%-50% | 20%     |

## User Experience

### Scenario 1: WiFi at Night (Ideal)

```
User goes to sleep at 11 PM
Phone on WiFi, charging

3:00 AM - App wakes up
3:01 AM - Checks for update (found v1.2!)
3:02 AM - Detects WiFi
3:03 AM - Downloads 4GB model (takes 5 min)
3:08 AM - Silent notification: "Alice updated!"
3:09 AM - Goes back to sleep

User wakes up at 7 AM
Opens app → New model ready! 🎉
```

### Scenario 2: Cellular at Night

```
User goes to sleep at 11 PM
Phone on cellular (no WiFi available)

3:00 AM - App wakes up
3:01 AM - Checks for update (found v1.2!)
3:02 AM - Detects cellular
3:03 AM - Sends notification with sound 🔔
3:03 AM - "New model available. Tap to download over cellular or wait for WiFi"

User wakes up at 7 AM
Sees notification
Taps "Wait for WiFi"
Connects to home WiFi
Download starts automatically
```

### Scenario 3: Low Battery

```
User goes to sleep at 11 PM
Phone at 15% battery (forgot to charge)

3:00 AM - App wakes up
3:01 AM - Checks battery (15% < 20%)
3:02 AM - Skips download to preserve battery
3:03 AM - Goes back to sleep

Next night (phone charging):
3:00 AM - Checks again
3:01 AM - Battery OK (85%)
3:02 AM - Downloads update successfully
```

## Technical Implementation

### iOS Background Tasks

```swift
// ios/App/App/AppDelegate.swift
import BackgroundTasks

func application(_ application: UIApplication,
                didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

    // Register background task
    BGTaskScheduler.shared.register(
        forTaskWithIdentifier: "com.adaptivefit.app.modelupdate",
        using: nil
    ) { task in
        self.handleModelUpdate(task: task as! BGProcessingTask)
    }

    // Schedule first task
    scheduleModelUpdate()

    return true
}

func scheduleModelUpdate() {
    let request = BGProcessingTaskRequest(identifier: "com.adaptivefit.app.modelupdate")
    request.requiresNetworkConnectivity = true
    request.requiresExternalPower = false // Allow on battery (we check manually)
    request.earliestBeginDate = Date(timeIntervalSinceNow: 3 * 60 * 60) // 3 hours from now

    try? BGTaskScheduler.shared.submit(request)
}
```

### Android WorkManager

```kotlin
// android/app/src/main/java/ModelUpdateWorker.kt
import androidx.work.*
import java.util.concurrent.TimeUnit

class ModelUpdateWorker(context: Context, params: WorkerParameters) : Worker(context, params) {
    override fun doWork(): Result {
        // Check battery
        // Check network
        // Download if conditions met
        return Result.success()
    }
}

// Schedule daily at 3 AM
val constraints = Constraints.Builder()
    .setRequiresBatteryNotLow(true)
    .setRequiredNetworkType(NetworkType.CONNECTED)
    .build()

val workRequest = PeriodicWorkRequestBuilder<ModelUpdateWorker>(24, TimeUnit.HOURS)
    .setConstraints(constraints)
    .setInitialDelay(calculateDelayTo3AM(), TimeUnit.MILLISECONDS)
    .build()

WorkManager.getInstance(context).enqueueUniquePeriodicWork(
    "model_update",
    ExistingPeriodicWorkPolicy.KEEP,
    workRequest
)
```

### Flutter WorkManager Implementation (Fresh app)

- **Worker registration**: `flutter_app/lib/main.dart` initializes `Workmanager` at startup and schedules the `nightly_federated` task via `scheduleNightlyFederatedTask()` so both Android and iOS/macOS BGTaskScheduler go through a single code path.
- **Task entrypoint**: `flutter_app/lib/core/background/nightly_federated_worker.dart` handles `callbackDispatcher`. Each execution now calls `runNightlyModelSync()` which mirrors the Svelte `backgroundModelUpdater` flow: refresh exercise/nutrition databases, ensure Alice assets, and throttle runs with SharedPreferences.
- **Model sync helper**: `flutter_app/lib/core/background/nightly_model_sync.dart` encapsulates Wi‑Fi gating, last-run timestamps, SharedModelStore handling, and federated uploader retries so that WorkManager runs stay aligned with the JS background updater semantics.

## Permissions Required

### iOS

- `UIBackgroundModes` → `fetch`, `processing`
- `BGTaskSchedulerPermittedIdentifiers`
- Notification permissions

### Android

- `WAKE_LOCK` - Wake device for background task
- `RECEIVE_BOOT_COMPLETED` - Restart tasks after reboot
- `FOREGROUND_SERVICE` - Keep task alive
- `ACCESS_NETWORK_STATE` - Check WiFi vs cellular
- `INTERNET` - Download model

## Testing

### Test Nighttime Check (Without Waiting)

```typescript
import { backgroundModelUpdater } from "$lib/services/ml/backgroundModelUpdater";

// Trigger immediate check (dev mode only)
await backgroundModelUpdater.triggerManualCheck();
```

### Simulate Network Conditions

```typescript
// In Chrome DevTools
// Network tab → Throttling → "Fast 3G" (simulate cellular)
// Or "Offline" (simulate no connection)

// Then trigger manual check
await backgroundModelUpdater.triggerManualCheck();
```

### Test Battery Threshold

```typescript
// Temporarily lower threshold to 90% (most phones are below)
backgroundModelUpdater.updateConfig({
  batteryThreshold: 90,
});

// Trigger check - should skip download due to battery
await backgroundModelUpdater.triggerManualCheck();
```

## Troubleshooting

### Updates Not Downloading

1. **Check Settings**
   - Settings → Model Updates
   - Ensure "Automatic Night Updates" is ON
   - Ensure "Auto-Download on WiFi" is ON

2. **Check Battery**
   - Battery must be > 20% (default)
   - Battery saver mode must be OFF

3. **Check Network**
   - Must be connected to WiFi (unless cellular is enabled)
   - Internet connection must be active

4. **Check Time**
   - Next check happens at 3 AM
   - Or trigger manual check in dev mode

### Cellular Prompts Not Appearing

1. **Check Notification Permissions**
   - Settings → App → Notifications → Allow

2. **Check Network Type**
   - Must be on cellular, not WiFi

3. **Check Logs**

   ```bash
   # iOS
   xcrun simctl spawn booted log stream --predicate 'processImagePath contains "YourApp"'

   # Android
   adb logcat | grep ModelUpdate
   ```

### Battery Draining Too Fast

**This shouldn't happen!** Background tasks are optimized for minimal battery usage:

- Only runs once per day (3 AM)
- Checks battery before starting
- Skips download if battery low
- Task completes in < 10 minutes

If battery drain occurs:

1. Disable automatic updates temporarily
2. Check for other apps using background data
3. Report issue with logs

## Best Practices

### For Users

✅ **DO:**

- Keep WiFi on at night (free, fast downloads)
- Charge phone overnight (allows downloads even at 20% battery)
- Enable automatic WiFi updates (hassle-free)

❌ **DON'T:**

- Enable cellular auto-download (unless unlimited data)
- Set battery threshold too high (prevents downloads)
- Disable notifications (you won't see cellular prompts)

### For Developers

✅ **DO:**

- Test on real devices (simulators don't have real battery/network)
- Use WorkManager (Android) / BGTaskScheduler (iOS)
- Respect user's data plan
- Provide clear notifications

❌ **DON'T:**

- Download large files without checking network type
- Drain battery with frequent checks
- Use polling instead of scheduled tasks
- Assume WiFi is always available

## Summary

**Goal:** Keep alice-human-fusion up-to-date without user intervention

**Method:** Smart nighttime downloads respecting battery, network, and user preferences

**Result:** Users wake up to the latest AI model, every week! 🎉

---

**Questions?** Check the code:

- `app/src/lib/services/ml/backgroundModelUpdater.ts` - Core logic
- `app/src/lib/components/settings/ModelUpdateSettings.svelte` - User settings
- `app/src/lib/init/backgroundTasks.ts` - Initialization

## Related
