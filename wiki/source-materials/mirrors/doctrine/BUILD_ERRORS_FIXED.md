---
title: BUILD_ERRORS_FIXED
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/BUILD_ERRORS_FIXED.md"]
updated: 2026-07-24
---

# Build Errors - Fixed

## Code Errors Fixed

### 1. ✅ Fixed `llama_free_context` → `llama_free`

- **Error**: `Cannot find 'llama_free_context' in scope`
- **Fix**: Changed to `llama_free(context)` (correct llama.cpp API)
- **Location**: Line 1624 in `LlamaEngine.swift`

### 2. ✅ Fixed `userMessage` optional binding

- **Error**: `Initializer for conditional binding must have Optional type, not 'String'`
- **Fix**: Changed `if let userMsg = userMessage` to `if !userMessage.isEmpty` (userMessage is String, not String?)
- **Location**: Line 2308 in `LlamaEngine.swift`

## Remaining Errors (Need Files Added to Xcode)

These errors will be resolved once you add the files to Xcode:

1. `Cannot find 'PromptBuilder' in scope` - Line 8112
2. `Cannot find type 'ContextResizer' in scope` - Line 8115
3. `Cannot find type 'TokenBudgeter' in scope` - Line 8118
4. `Cannot find type 'ContextResizerDeviceTier' in scope` - Line 1388
5. `Cannot find 'ContextResizer' in scope` - Line 1406
6. `Cannot find 'TokenBudgeter' in scope` - Line 1413

## Solution: Add Files to Xcode Project

**Steps:**

1. Open `flutter_app/ios/Runner.xcworkspace` in Xcode
2. Right-click on **`Runner`** folder in Project Navigator
3. Select **"Add Files to Runner..."**
4. Navigate to `flutter_app/ios/Runner/` directory
5. Select both files:
   - ✅ `PromptBuilder.swift`
   - ✅ `ContextResizer.swift`
6. **CRITICAL Settings:**
   - ✅ **"Add to targets: Runner"** - MUST be checked
   - ❌ **"Copy items if needed"** - Should be UNCHECKED
7. Click **"Add"**

**Verify:**

- Files appear in Project Navigator under `Runner`
- Go to **Build Phases → Compile Sources**
- Both files should be listed

**Then:**

- Clean build: **Product → Clean Build Folder** (Shift+Cmd+K)
- Build: **Product → Build** (Cmd+B)

## Summary

✅ **Code errors fixed** - All Swift syntax errors resolved
⏳ **Files need to be added** - Add `PromptBuilder.swift` and `ContextResizer.swift` to Xcode project

Once files are added, the build should succeed!

## Related
