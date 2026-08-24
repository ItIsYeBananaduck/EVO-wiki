---
title: BUILD_TROUBLESHOOTING
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/BUILD_TROUBLESHOOTING.md"]
updated: 2026-07-24
---

# Build Troubleshooting Guide

## Issue: Build Failing After Adding Optimization Files

### Step 1: Verify Files Exist

```bash
ls -la flutter_app/ios/Runner/PromptBuilder.swift
ls -la flutter_app/ios/Runner/ContextResizer.swift
```

Both files should exist.

### Step 2: Add Files to Xcode Project

**Method 1: Via Xcode UI (Recommended)**

1. Open `flutter_app/ios/Runner.xcworkspace` in Xcode
2. In Project Navigator, right-click on the **`Runner`** folder (not the project root)
3. Select **"Add Files to Runner..."**
4. Navigate to `flutter_app/ios/Runner/` directory
5. Select both files:
   - `PromptBuilder.swift`
   - `ContextResizer.swift`
6. **CRITICAL**: Make sure these checkboxes are set:
   - ✅ **"Add to targets: Runner"** (MUST be checked)
   - ❌ **"Copy items if needed"** (should be UNCHECKED - files are already in the right place)
7. Click **"Add"**

**Method 2: Drag and Drop**

1. Open `flutter_app/ios/Runner.xcworkspace` in Xcode
2. Open Finder and navigate to `flutter_app/ios/Runner/`
3. Drag `PromptBuilder.swift` and `ContextResizer.swift` into the `Runner` folder in Xcode's Project Navigator
4. In the dialog, ensure **"Add to targets: Runner"** is checked
5. Click **"Finish"**

### Step 3: Verify Files Are in Build Phases

1. In Xcode, select the **Runner** project (blue icon) in Project Navigator
2. Select the **Runner** target
3. Go to **"Build Phases"** tab
4. Expand **"Compile Sources"**
5. Verify both files are listed:
   - `PromptBuilder.swift`
   - `ContextResizer.swift`
6. If missing, click the **"+"** button and add them

### Step 4: Clean and Rebuild

1. In Xcode: **Product → Clean Build Folder** (Shift+Cmd+K)
2. **Product → Build** (Cmd+B)

### Step 5: If Still Failing - Get Exact Error

Please share the **exact error message** from Xcode. Common errors:

**Error: "No such module 'PromptBuilder'" or "Cannot find 'PromptBuilder' in scope"**

- Files not added to Xcode project (follow Step 2)

**Error: "Use of unresolved identifier 'ContextResizerDeviceTier'"**

- Files not compiled together (check Build Phases)

**Error: "Duplicate symbol"**

- Files added twice (remove duplicates from Build Phases)

**Error: "Missing required module"**

- Clean build folder and rebuild

### Quick Verification Script

Run this to check if files are in the project:

```bash
cd flutter_app/ios
grep -r "PromptBuilder.swift\|ContextResizer.swift" Runner.xcodeproj/project.pbxproj
```

If you see the filenames, they're in the project. If not, they need to be added.

### Alternative: If You Can't Add Files Manually

If you're having trouble, you can temporarily comment out the code that uses these classes:

1. In `LlamaEngine.swift`, find:

   ```swift
   private let promptBuilder = PromptBuilder()
   private var contextResizer: ContextResizer?
   private var tokenBudgeter: TokenBudgeter?
   ```

2. Comment them out and the code that uses them (this will disable optimizations but allow build)

3. Share the exact error message so I can help fix it properly

## Need Help?

Please share:

1. The **exact error message** from Xcode's Issue Navigator (red X icon)
2. Whether files appear in "Compile Sources" (Build Phases)
3. Screenshot of Xcode showing the error (if possible)

## Related
