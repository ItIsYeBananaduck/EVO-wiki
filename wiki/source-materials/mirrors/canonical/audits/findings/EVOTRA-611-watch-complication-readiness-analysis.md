---
title: "EVOTRA-611 Watch Complication Readiness Analysis"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-06-18
---

# EVOTRA-611 Watch Complication Readiness Analysis

## Summary

Linear intake was requested as `EVTRA-611`; exact lookup failed. Linear search resolved the intended issue as `EVOTRA-611`, "Investigate Apple Watch complication readiness stuck at stale value."

The readiness calculation path is likely healthy. Current code publishes a canonical dashboard snapshot, writes iOS widget keys, sends stats to the watch through WatchConnectivity, and the watch app writes complication-readable `widget_*` keys before requesting WidgetKit timeline reloads. The stale complication symptom is therefore most likely in the watch complication refresh/package boundary, not in the readiness score calculation.

## Evidence

- `flutter_app/lib/services/canonical_dashboard_metrics_service.dart` centralizes metric fan-out in `publishSnapshot`: cache update, App Group persistence, widget key sync, timeline reload, watch sync, then UI notification.
- `_syncToWidget` writes `widget_readiness_score`, `widget_readiness_state`, `widget_snapshot_version`, `widget_status`, and `widget_last_updated`.
- `flutter_app/ios/Runner/AppDelegate.swift` handles `reloadComplications` by calling `WidgetCenter.shared.reloadAllTimelines()` on iOS, but the comment correctly notes ClockKit/watch complications are handled by the watch app.
- `LiveWorkoutWearableBridge.sendStats` sends normal WatchConnectivity messages, updates application context, and attempts `transferCurrentComplicationUserInfo` when `WCSession.isComplicationEnabled`.
- `flutter_app/ios/watchApp Watch App/MiniAliceSession.swift` receives stats, rejects stale versions, writes `widget_*` keys to `group.biz.lsctech.adaptivefit`, verifies persistence, and calls `WidgetCenter.shared.reloadAllTimelines()` plus `reloadTimelines(ofKind:)`.
- `flutter_app/ios/watchApp Watch App/EvoWatchComplication/EvoWatchComplications.swift` reads those same `widget_*` keys from the same app group and renders readiness by family.
- Runner, watch app, and complication extension entitlements all include `group.biz.lsctech.adaptivefit`.

## Likely Root Cause

The highest-probability root cause is that the watch complication extension is not being refreshed from the same active watch app data path that the watch app UI uses. The code is structured to make that work, but there are two concrete risks:

1. `transferCurrentComplicationUserInfo` is gated by `WCSession.isComplicationEnabled`. If the installed complication is not considered enabled, the phone will not wake the complication path even though the watch app can receive regular stats later.
2. Xcode synchronized-folder membership for `EvoWatchComplicationExtension` has exception entries for the complication Swift files under `flutter_app/ios/Runner.xcodeproj/project.pbxproj`. Target membership should be verified in Xcode/build settings to ensure the installed extension is the current source, not a stale or partially packaged extension.

Secondary risk: existing watch tests are stale. `flutter_app/ios/watchApp Watch AppTests/watchApp_Watch_AppTests.swift` asserts older symbols such as `writeStrainDataForComplications`, which now exist in `flutter_app/ios/WatchApp/MiniAliceSession.swift`, while the active project target appears to use `flutter_app/ios/watchApp Watch App/MiniAliceSession.swift`.

## Affected Components

- `flutter_app/lib/services/canonical_dashboard_metrics_service.dart`
- `flutter_app/lib/services/watch_sync_service.dart`
- `flutter_app/ios/Runner/AppDelegate.swift`
- `flutter_app/ios/watchApp Watch App/MiniAliceSession.swift`
- `flutter_app/ios/watchApp Watch App/EvoWatchComplication/EvoWatchComplications.swift`
- `flutter_app/ios/Runner.xcodeproj/project.pbxproj`
- `flutter_app/ios/watchApp Watch AppTests/watchApp_Watch_AppTests.swift`

## Recommended Implementation Plan

1. Add observability around complication delivery:
   - Log when `isComplicationEnabled` is false and include activation state, pairing, install state, and latest snapshot version.
   - Include `snapshotVersion` and readiness in every phone-to-watch complication payload log.
   - Add a watch-side log/event after `reloadTimelines(ofKind:)` for each kind.

2. Make refresh delivery more resilient:
   - Keep `transferCurrentComplicationUserInfo` when enabled.
   - When not enabled, still ensure application context and queued user info carry the latest snapshot and schedule a watch-side reload when the watch receives it.
   - Consider request/replay behavior when the watch app activates and sees a stale or missing complication snapshot.

3. Verify and repair target membership:
   - Confirm `EvoWatchComplications.swift` and `EvoWatchComplicationBundle.swift` are compiled into `EvoWatchComplicationExtension`.
   - Confirm `MiniAliceSession.swift` under `watchApp Watch App` is the active watch app source and remove/retire the older duplicate path if it is no longer used.

4. Replace stale tests with current-source tests:
   - Assert `didReceiveUserInfo` calls the current stats handling path.
   - Assert `writeStatsToAppGroup` writes all `widget_*` keys used by `EvoWatchComplications.swift`.
   - Assert `reloadComplicationTimelines` calls both `reloadAllTimelines` and the configured complication kinds.

5. Validate on device or paired simulator:
   - Publish a snapshot with readiness different from 23.
   - Confirm phone widget updates.
   - Confirm watch app displays the same version/readiness.
   - Confirm watch app group `widget_snapshot_version` and `widget_readiness_score` update.
   - Confirm complication timeline logs read the same version/readiness and render it.

## Blockers

No repository blocker prevents implementation planning. Root-cause confirmation needs a paired watch runtime or device logs because WidgetKit complication refresh behavior is runtime-delivery dependent.

## Confidence

88%. The code evidence narrows the issue to watch complication delivery/packaging. Confidence stops short of certainty because confirming `isComplicationEnabled` behavior and extension packaging requires runtime or Xcode target inspection beyond static source reads.