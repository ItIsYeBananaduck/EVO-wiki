---
title: XCODE_ARCHIVE_STEPS
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/XCODE_ARCHIVE_STEPS.md
updated: 2026-07-24
---

# Xcode Archive & TestFlight Upload - Manual Steps

## ✅ Xcode is Now Open

I've opened the workspace in Xcode. Follow these steps:

### Step 1: Verify Settings (in Xcode)

1. **Select "Runner"** in the left sidebar (project navigator)
2. **Select "Runner" target** (under TARGETS)
3. **Go to "Signing & Capabilities" tab**
4. **Verify**:
   - ✅ Team is selected
   - ✅ Bundle Identifier is correct (com.evo.evotraining)
   - ✅ "Automatically manage signing" is checked

### Step 2: Select Device Target

1. **In the top toolbar**, click the device selector (next to the Run/Stop buttons)
2. **Select "Any iOS Device"** (NOT a simulator, NOT a specific device)
   - Should show: "Any iOS Device (arm64)"

### Step 3: Archive

1. **Product → Archive** (or press `Cmd+Shift+B`, then click "Archive")
2. **Wait** for archive to complete (5-10 minutes)
   - You'll see progress in Xcode
   - Build warnings are okay (not errors)
   - Build must complete successfully

### Step 4: Upload to TestFlight

1. **Window → Organizer** (or click "Organizer" if prompted)
2. **Select your archive** (latest one, should show today's date)
3. **Click "Distribute App"** button
4. **Select "TestFlight & App Store"** → **Next**
5. **Choose upload options**:
   - ✅ Upload your app's symbols (recommended)
   - ✅ Manage version and build number automatically
   - Click **Next**
6. **Review**:
   - App information
   - Version number
   - Build number
   - Click **Next**
7. **Click "Upload"**
8. **Wait** for upload to complete (5-15 minutes)

### Step 5: Wait for Processing

1. **Open App Store Connect** (appstoreconnect.apple.com) in browser
2. **Go to your app → TestFlight**
3. **Wait** for processing (10-30 minutes typically)
4. **Status will change** from "Processing" to "Ready to Test"

### Step 6: Test!

1. **Install TestFlight** on iPhone (if not already)
2. **Accept TestFlight invite** (check email)
3. **Install app** from TestFlight
4. **Open app** and check diagnostics:
   - Long press in Alice chat
   - Tap "Diagnostics"
   - Go to "Context & Tools" tab
   - Verify context window shows 2048-4096 (depending on device)
   - Check tool calling status

## 🔍 Troubleshooting

### If Archive Fails:

- Check Xcode error messages
- Verify signing certificates
- Try cleaning: Product → Clean Build Folder (`Cmd+Shift+K`)
- Try again: Product → Archive

### If Upload Fails:

- Check internet connection
- Verify App Store Connect access
- Check signing certificates are valid
- Try again from Organizer

### If Processing Fails:

- Check App Store Connect for error messages
- Verify app compliance
- Check app version/build number

## 📊 What to Check in TestFlight

Once processing completes:

### In Diagnostics ("Context & Tools" tab):

- **Context Window**:
  - Device Tier: high/midHigh/mid/etc.
  - Current Context: 2048-4096 (should be 2x old value)
  - Context Size Increase: "2x (upgraded)"
- **Tool Calling**:
  - Enabled: true
  - Total Tools Defined: 17+ (number of tools)
  - Token Savings: ~70-85%

### Functional Tests:

- Start long conversation (20+ messages)
- Alice should remember earlier messages
- Try tool calling: "Go to workouts" or "Create a plan"
- No crashes

## 🎯 Success Indicators

✅ Archive completes successfully
✅ Upload completes successfully
✅ Processing completes (status = "Ready to Test")
✅ App installs on iPhone via TestFlight
✅ Diagnostics show correct context window
✅ Long conversations work
✅ Tool calling works

Good luck! 🚀

## Related

^[source-materials/mirrors/doctrine/XCODE_ARCHIVE_STEPS.md]
