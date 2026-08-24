---
title: MACOS_SETUP_GUIDE
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/MACOS_SETUP_GUIDE.md"]
updated: 2026-07-24
---

# macOS App Setup Guide - Complete ✅

## 🍎 What I've Configured

### 1. Bundle Identifier ✅

- **Updated**: `com.example.flutterApp` → `com.evo.evotraining`
- **File**: `macos/Runner/Configs/AppInfo.xcconfig`
- **Matches**: iOS app bundle identifier for consistency

### 2. App Name ✅

- **Updated**: `flutter_app` → `git-fit`
- **File**: `macos/Runner/Configs/AppInfo.xcconfig`
- **Result**: Proper app name in macOS menu bar and window title

### 3. OAuth Deep Link Support ✅

- **Added**: `CFBundleURLTypes` to `macos/Runner/Info.plist`
- **Scheme**: `com.evo.evotraining://auth-callback`
- **Purpose**: OAuth callbacks work on macOS too

### 4. CocoaPods Integration ✅

- **Installed**: All macOS dependencies
- **Fixed**: Configuration warnings with proper includes
- **Status**: Ready to build

## 🚀 How to Use the macOS App

### Build and Run

```bash
# From flutter_app directory
flutter run -d macos --debug

# Or build for distribution
flutter build macos --release
```

### Xcode Development

1. **Open**: `macos/Runner.xcworkspace` (not .xcodeproj)
2. **Target**: Runner
3. **Bundle Identifier**: `com.evo.evotraining`
4. **Deployment Target**: macOS 10.15 or later

## 🔧 Key Configurations

### App Capabilities

The macOS app supports:

- ✅ OAuth (Google/Apple) with deep links
- ✅ Local storage (SQLite)
- ✅ File picking
- ✅ Secure storage
- ✅ Network connectivity
- ✅ Speech to text
- ✅ TTS (Text to Speech)
- ✅ WebRTC (for future features)
- ✅ ONNX Runtime (for AI models)

### OAuth on macOS

OAuth works slightly differently on macOS:

1. **In-app browser** opens for authentication
2. **Deep link** returns to the app after success
3. **URL scheme**: `com.evo.evotraining://auth-callback`
4. **Same Supabase configuration** as iOS

### Development Notes

- **Hot reload**: Works on macOS
- **Debugging**: Use Xcode debugger or Flutter inspector
- **File system**: Full macOS file system access
- **Permissions**: macOS will request permissions as needed

## 📱 Testing OAuth on macOS

To test OAuth on macOS:

1. Run the app: `flutter run -d macos`
2. Click "Continue with Google" or "Continue with Apple"
3. Browser opens → Complete authentication
4. Returns to app automatically
5. User should be logged in

## 🎯 Next Steps

### Distribution

When ready to distribute:

1. **Code Signing**: Set up Apple Developer certificate
2. **Notarization**: Required for distribution outside App Store
3. **App Store**: Optional - can distribute directly

### Enhancements

Consider adding:

- **Menu bar integration**
- **Dock icon customization**
- **Keyboard shortcuts**
- **Native macOS dialogs**
- **File association** (e.g., .csv workout files)

## 🔍 Troubleshooting

### Common Issues

1. **Build fails**: Clean and retry `flutter clean && flutter pub get`
2. **OAuth not working**: Check Supabase redirect URLs include macOS scheme
3. **Deep links not working**: Verify URL scheme in Info.plist
4. **Permissions denied**: Grant permissions in System Preferences

### Debug Commands

```bash
# Check available devices
flutter devices

# Clean build
flutter clean

# Get dependencies
flutter pub get

# Build only
flutter build macos --debug
```

## ✅ Summary

Your macOS app is now:

- ✅ Properly configured with correct bundle ID
- ✅ Ready for OAuth authentication
- ✅ Integrated with all required dependencies
- ✅ Open in Xcode for further development

The macOS app should work identically to the iOS app for core functionality, with the added benefits of desktop features like larger screen space and full file system access.

## Related
