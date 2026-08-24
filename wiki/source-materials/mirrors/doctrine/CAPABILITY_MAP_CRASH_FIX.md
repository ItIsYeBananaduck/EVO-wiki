---
title: CAPABILITY_MAP_CRASH_FIX
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CAPABILITY_MAP_CRASH_FIX.md"]
updated: 2026-07-24
---

# Capability Map Crash Fix

## Issue

`EXC_BAD_ACCESS (code=2, address=0x2000000076)` crash when loading capability map.

## Fixes Applied

### 1. Feature Flag

Added `capabilityMapEnabled` flag at line ~5198 in `LlamaEngine.swift`:

```swift
private let capabilityMapEnabled = true
```

**To disable capability map loading temporarily**, change to `false`.

### 2. Multiple File Loading Methods

- Primary: `Bundle.main.path(forResource:ofType:)`
- Fallback: Direct path from `Bundle.main.resourcePath`

### 3. Memory Safety

- Wrapped JSON parsing in `autoreleasepool`
- Wrapped filtering in `autoreleasepool`
- Create dictionary copies to ensure memory ownership
- Safe dictionary key iteration (copy keys to array first)

### 4. Defensive Dictionary Access

- Check `keys.contains()` before accessing
- Safe type casting with multiple guards
- Early returns on failures

### 5. Error Handling

- All file I/O wrapped in try-catch
- All JSON parsing wrapped in try-catch
- Graceful degradation (returns nil, continues without map)

## Testing

### If Still Crashing:

1. **Disable capability map temporarily:**

   ```swift
   private let capabilityMapEnabled = false
   ```

   This will skip all capability map loading.

2. **Verify file is in bundle:**
   - Check Xcode project: `flutter_app/ios/Runner/alice_capability_map.json`
   - Ensure it's added to target in "Build Phases" > "Copy Bundle Resources"

3. **Check crash log:**
   - Look for exact line number in stack trace
   - Check if it's in `loadRelevantCapabilities`, `formatCapabilityMap`, or `createCompactCapabilityMap`

## File Location

The capability map should be at:

- Source: `flutter_app/ios/Runner/alice_capability_map.json`
- Bundle: Included in app bundle at runtime

## Next Steps if Still Crashing

1. Set `capabilityMapEnabled = false` to confirm it's the capability map causing the crash
2. If disabling fixes it, the issue is in file loading/parsing
3. If it still crashes, the issue is elsewhere

## Related
