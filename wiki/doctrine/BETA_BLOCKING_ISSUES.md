---
title: BETA_BLOCKING_ISSUES
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/BETA_BLOCKING_ISSUES.md
updated: 2026-07-24
---

# Beta Blocking Issues - Audit Report

## Critical Issues Found

### 1. **Watch Complications Not Selectable**

**Root Cause**: Missing Info.plist in watchApp target - complications need CLKComplicationPrincipalClass entry
**Impact**: Complications don't appear in watch face picker
**Fix**: Create Info.plist with proper complication configuration

### 2. **Widgets Showing Placeholder Data**

**Root Cause**: Widget data models have hardcoded placeholder fallbacks
**Files**:

- `ios/EvoWidget/EvoWidget.swift` - lines 22-30, 40-46, 56-62 (placeholder data)
- Widget provider returns `.placeholder` when no data exists
  **Impact**: Users see fake data (75 strain, 3 workouts, etc.) instead of "No Data"
  **Fix**: Remove placeholder fallbacks, show explicit "No Data" state

### 3. **No Readiness Data**

**Root Cause**: Readiness score not being calculated or displayed
**Missing**:

- Readiness calculation from strain records
- Readiness display in widgets/complications
- Readiness sync to watch
  **Impact**: Key metric missing from all surfaces
  **Fix**: Calculate readiness as (100 - compositeStrainScore) and propagate everywhere

### 4. **Watch Not Syncing with Phone**

**Root Causes**:
a) WatchConnectivity session activated but no automatic data push on app launch
b) Dashboard tab sends data but only when explicitly loaded
c) No periodic sync mechanism
d) Widget/complication reload not triggered after data updates
**Impact**: Watch shows stale/no data even when phone has fresh data
**Fix**:

- Trigger sync on app launch (HomeScreen initState)
- Trigger sync when returning to foreground
- Ensure widget reload after every data write

### 5. **Missing CLKComplicationServer Reload**

**Root Cause**: iOS AppDelegate calls WidgetCenter.shared.reloadAllTimelines() but not CLKComplicationServer
**Impact**: Watch complications don't update even when data changes
**Fix**: Add CLKComplicationServer.sharedInstance().reloadTimelines(for:) calls

## Data Flow Analysis

### Current Flow (Broken)

1. Phone: HomeScreen loads → calculates metrics
2. Phone: Writes to UserDefaults (app group)
3. Phone: Calls WidgetCenter.reloadAllTimelines() ✓
4. Phone: Does NOT call CLKComplicationServer ✗
5. Phone: Does NOT push to watch via WatchConnectivity ✗
6. Watch: Never receives update ✗
7. Widgets: Show placeholder data ✗

### Required Flow (Fixed)

1. Phone: HomeScreen loads → calculates metrics
2. Phone: Writes to UserDefaults (app group)
3. Phone: Calls WidgetCenter.reloadAllTimelines() ✓
4. Phone: Calls CLKComplicationServer.reloadTimelines() ✓ (NEW)
5. Phone: Pushes stats to watch via WatchConnectivity ✓ (NEW)
6. Watch: Receives update, writes to app group ✓ (NEW)
7. Watch: Reloads complications ✓ (NEW)
8. Widgets: Show real data or explicit "No Data" ✓

## Files Requiring Changes

### High Priority (Beta Blockers)

1. `ios/watchApp Watch App/Info.plist` - CREATE (complication registration)
2. `ios/EvoWidget/EvoWidget.swift` - MODIFY (remove placeholders)
3. `ios/Runner/AppDelegate.swift` - MODIFY (add CLKComplicationServer reload)
4. `lib/features/home/presentation/home_screen.dart` - MODIFY (trigger sync on launch)
5. `lib/services/widget_data_service.dart` - MODIFY (add CLKComplicationServer support)

### Medium Priority (Data Quality)

6. `ios/watchApp Watch App/Complications/EvoWatchComplications.swift` - MODIFY (handle no-data state)
7. `ios/watchApp Watch App/MiniAliceSession.swift` - MODIFY (write to app group on stats update)
8. `lib/features/home/presentation/home_screen.dart` - MODIFY (calculate readiness properly)

## Implementation Order

1. Create watchApp Info.plist with complication registration
2. Add CLKComplicationServer reload to AppDelegate
3. Remove placeholder fallbacks from widgets
4. Add watch sync trigger on HomeScreen launch
5. Implement readiness calculation
6. Add app group write on watch stats receive
7. Test end-to-end data flow

## Related

^[source-materials/mirrors/doctrine/BETA_BLOCKING_ISSUES.md]
