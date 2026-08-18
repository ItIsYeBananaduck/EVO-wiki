---
title: BUILD_FIX_INSTRUCTIONS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/BUILD_FIX_INSTRUCTIONS.md"]
updated: 2026-07-24
---

# Build Fix Instructions

## Issue

The build is failing because the new Swift files need to be added to the Xcode project.

## Solution

### Option 1: Add Files via Xcode (Recommended)

1. Open `flutter_app/ios/Runner.xcworkspace` in Xcode
2. Right-click on the `Runner` folder in the Project Navigator
3. Select "Add Files to Runner..."
4. Navigate to `flutter_app/ios/Runner/` and select:
   - `PromptBuilder.swift`
   - `ContextResizer.swift`
5. Make sure "Copy items if needed" is **unchecked** (files are already in the right location)
6. Make sure "Add to targets: Runner" is **checked**
7. Click "Add"

### Option 2: Add Files via Command Line

If you prefer command line, you can add the files to the Xcode project using `xcodeproj` gem or manually edit the project file, but Option 1 is easier.

### Option 3: Verify Files Are in Project

If files are already added but build still fails:

1. In Xcode, select the project in Project Navigator
2. Select the "Runner" target
3. Go to "Build Phases" → "Compile Sources"
4. Verify both `PromptBuilder.swift` and `ContextResizer.swift` are listed
5. If missing, add them using the "+" button

## Verification

After adding the files, the build should succeed. The Swift files have been verified to type-check correctly:

- ✅ `PromptBuilder.swift` - No syntax errors
- ✅ `ContextResizer.swift` - No syntax errors
- ✅ `LlamaEngine.swift` - No syntax errors (modified)

## Common Build Errors Fixed

- ✅ DeviceTier enum conflict resolved (renamed to `ContextResizerDeviceTier`)
- ✅ All type references corrected
- ✅ Variable scoping verified (`shouldClearCache` is accessible)

## If Build Still Fails

Please share the specific error message from Xcode, and I can help debug further.

## Related

^[{src_rel}]
