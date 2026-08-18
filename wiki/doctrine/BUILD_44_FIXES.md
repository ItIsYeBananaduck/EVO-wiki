---
title: BUILD_44_FIXES
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/BUILD_44_FIXES.md"]
updated: 2026-07-24
---

# Build 44 Fixes

## Issues Found and Fixed

### ✅ Fixed: Version Updated

- Updated `pubspec.yaml` version from `1.0.0+42` to `1.0.0+44`

### ✅ Fixed: EvoWidget Version Mismatch

- **Problem**: EvoWidget had hardcoded `CURRENT_PROJECT_VERSION = 1`, causing validation error:
  ```
  warning: The CFBundleVersion of an app extension ('1') must match that of its containing parent app ('44').
  ```
- **Fix**: Updated all EvoWidget build configurations (Debug, Release, Profile) to use `$(FLUTTER_BUILD_NUMBER)` instead of hardcoded `1`

### ⚠️ Remaining Issues (Need Manual Fix or Investigation)

#### 1. Encryption Info Issue

```
The binary is invalid. The encryption info in the LC_ENCRYPTION_INFO load command is either missing or invalid, or the binary is already encrypted.
```

**Possible Causes:**

- Archive is signed with "Apple Development" instead of "Apple Distribution"
- This happens when archiving via command line

**Solution:**

- Use Xcode Organizer to distribute the archive instead
- Xcode will automatically use the correct distribution certificate
- Or configure signing in Xcode to use Distribution profiles for Release builds

#### 2. sqlite3arm64ios_sim.framework OS Version Issue

```
Invalid Bundle. The bundle Runner.app/Frameworks/sqlite3arm64ios_sim.framework does not support the minimum OS Version specified in the Info.plist.
```

**Possible Causes:**

- Simulator framework is being included in device builds
- Framework doesn't support iOS 16.0 minimum deployment target

**Solution:**

- Check `sqflite` plugin configuration
- May need to exclude simulator architectures from release builds
- Check Podfile for architecture-specific framework exclusions

#### 3. Missing dSYM Files

Missing debug symbols for:

- Pods_Runner.framework
- llama.framework
- sqlite3arm64ios.framework
- sqlite3arm64ios_sim.framework

**Solution:**

- These are warnings, not errors
- Can be fixed by ensuring DEBUG_INFORMATION_FORMAT is set correctly
- Or by manually generating dSYMs if needed for crash reporting

## Next Steps

1. **Rebuild the archive** with the fixed version numbers
2. **Use Xcode Organizer** to distribute to TestFlight (handles signing automatically)
3. **If issues persist**, investigate:
   - sqlite framework architecture exclusions
   - Signing configuration for Release builds
   - Framework deployment targets

## Testing

After fixes, verify:

- [x] Version is 44 in pubspec.yaml
- [x] EvoWidget version matches app version
- [ ] Archive signs with Distribution certificate
- [ ] No simulator frameworks in device build
- [ ] Upload succeeds to App Store Connect

## Related

^[{src_rel}]
