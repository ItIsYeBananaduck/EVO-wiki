---
title: IOS_MIGRATION_STATUS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/IOS_MIGRATION_STATUS.md"]
updated: 2026-07-24
---

# iOS Migration Status - Flutter App

## ✅ Completed Configuration

### 1. App Branding & Identity

- **Bundle ID**: `biz.lsctech.adaptivefit` (matches existing Capacitor app)
- **App Name**: EVOtraining
- **Display Name**: EVOtraining

### 2. iOS Deployment Target

- **Minimum iOS Version**: 15.0 (updated from 13.0)
- Configured in `ios/Podfile`
- Applied to all CocoaPods dependencies

### 3. Permissions Configured

All necessary iOS permissions added to `Info.plist`:

- ✅ HealthKit (read & write)
- ✅ Camera access
- ✅ Photo library (read & add)
- ✅ Microphone
- ✅ Motion & fitness
- ✅ Location (when in use)

### 4. Entitlements Configured

Added to `Runner.entitlements`:

- ✅ HealthKit access
- ✅ App Groups (`group.biz.lsctech.adaptivefit`)
- ✅ Keychain sharing

### 5. Environment Configuration

- ✅ Created `.env` file with Supabase credentials
- ✅ Configured API endpoints
- ✅ Set up federated learning server URL
- ✅ Background task configuration

### 6. Build Configuration

- ✅ Updated `Debug.xcconfig` with bundle ID
- ✅ Updated `Release.xcconfig` with bundle ID
- ✅ CocoaPods dependencies installed successfully
- ✅ Created `build-ios.sh` script for easy building

## ⚠️ Known Issues

### Build Errors

The app has compilation errors that need to be fixed before it can build:

1. **Missing Widget**: `GlassAlertDialog` not defined
   - Location: `lib/features/home/presentation/new_workout_screen.dart:1520`
   - Fix: Implement or import the missing widget

2. **Color API Issue**: `shade700` getter doesn't exist
   - Location: `lib/features/pose/presentation/pose_debug_screen.dart:193`
   - Fix: Update to use correct Material color API

3. **Image Package API Changes**: Multiple issues with `image` package v4.x
   - `setPixelRgba()` signature changed (needs 6 args, not 5)
   - `getRed()`, `getGreen()`, `getBlue()` methods removed
   - `getPixel()` returns `Pixel` object, not `int`
   - Fix: Update to use new `image` package API

### iOS SDK Issue

- Xcode is looking for iOS 26.2 SDK
- Available SDKs: iOS 18.6, iOS 26.0
- This may require updating Xcode or adjusting build settings

## 📝 Next Steps

### Immediate (Fix Build Errors)

1. Fix `GlassAlertDialog` widget reference
2. Update color API usage
3. Update `image` package usage to v4.x API
4. Test build after fixes

### Short Term (iOS Deployment)

1. Connect a physical iOS device or start a simulator
2. Configure code signing in Xcode
3. Test app on device/simulator
4. Verify all features work correctly

### Medium Term (Feature Parity)

1. Migrate remaining features from Capacitor app
2. Test HealthKit integration
3. Test background tasks
4. Test Watch app connectivity
5. Verify federated learning works

### Long Term (Production)

1. Set up TestFlight for beta testing
2. Update CI/CD pipeline for Flutter builds
3. Create App Store screenshots
4. Submit to App Store review

## 🚀 How to Build (Once Errors Fixed)

### Option 1: Using the build script

```bash
cd flutter_app
./build-ios.sh
```

### Option 2: Using Flutter directly

```bash
cd flutter_app
/Users/user287043/development/flutter/bin/flutter build ios --release --no-codesign
```

### Option 3: Using Xcode

```bash
cd flutter_app
open ios/Runner.xcworkspace
```

Then build and run from Xcode (⌘R).

## 📚 Documentation Created

- ✅ `SETUP.md` - Comprehensive setup guide
- ✅ `build-ios.sh` - Build automation script
- ✅ `.env` - Environment configuration (gitignored)
- ✅ This status document

## 🔧 Files Modified

1. `ios/Runner/Info.plist` - Added permissions
2. `ios/Runner/Runner.entitlements` - Added capabilities
3. `ios/Flutter/Debug.xcconfig` - Added bundle ID
4. `ios/Flutter/Release.xcconfig` - Added bundle ID
5. `ios/Podfile` - Updated iOS deployment target
6. `pubspec.yaml` - Updated app description
7. `.env` - Created with configuration

## ✨ Summary

The Flutter app has been successfully configured to match the existing EVOtraining (adaptive fit) iOS app. All necessary permissions, entitlements, and build settings are in place. However, there are compilation errors in the Dart code that need to be fixed before the app can build and run. Once these are resolved, the app should be ready for testing on iOS devices.

The configuration work is complete - the remaining work is fixing the code errors and testing.

## Related
