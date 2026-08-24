---
title: MLX_Swift_Setup_Instructions
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/MLX_Swift_Setup_Instructions.md
updated: 2026-07-24
---

# MLX Swift Setup Instructions

## Step 1: Open Xcode Project

```bash
cd /Users/user287043/Documents/git-fit/flutter_app/ios
open Runner.xcworkspace
```

**Important**: Open `.xcworkspace` (not `.xcodeproj`) because of CocoaPods.

---

## Step 2: Add MLX Swift Package (Local Path Method - Recommended)

**Status**: ✅ MLX Swift has been cloned locally with all submodules to `flutter_app/ios/Packages/mlx-swift`

### Add the Package via Local Path:

1. **Open Xcode**:

   ```bash
   cd flutter_app/ios
   open Runner.xcworkspace
   ```

2. In Xcode, go to **File > Add Package Dependencies...**

3. At the bottom left, click **"Add Local..."** (NOT the URL field)

4. Navigate to:

   ```
   flutter_app/ios/Packages/mlx-swift
   ```

   Or use the absolute path:

   ```
   /Users/user287043/Documents/git-fit/flutter_app/ios/Packages/mlx-swift
   ```

5. Click **Add Package**

6. When prompted to choose products, select:
   - ✅ `MLX` (required - core framework)
   - ✅ `MLXNN` (neural network operations - needed for models)
   - ✅ `MLXRandom` (random number generation)
   - ✅ `MLXOptimizers` (optimizers - needed for training/LoRA)

7. For the target, select **Runner**

8. Click **Add Package**

9. Wait for Xcode to resolve dependencies (may take 1-2 minutes)

### Verify Installation:

1. Check that `MLXEngine.swift` can now import MLX:

   ```swift
   #if canImport(MLX)
   import MLX
   import MLXNN
   #endif
   ```

2. Build the project: **⌘B** (Product > Build)

---

## Alternative: Remote URL (If Local Path Fails)

If you prefer to use the remote URL instead:

1. Go to **File > Add Package Dependencies...**

2. Enter:

   ```
   https://github.com/ml-explore/mlx-swift.git
   ```

3. Select version/branch (try `main` or latest release)

4. If you get "invalid response", try:
   - **File > Packages > Reset Package Caches**
   - Try again with URL: `https://github.com/ml-explore/mlx-swift` (no `.git`)

---

## Updating the Local Package Later

To update MLX Swift to the latest version:

```bash
cd flutter_app/ios/Packages/mlx-swift
git pull origin main
git submodule update --init --recursive
```

Then rebuild in Xcode.

---

## Step 3: Add MLX LLM Package (for text generation)

**Note**: MLX LLM might not have a separate package. The LLM functionality may be in the main `mlx-swift` package or require building from source.

If MLX LLM isn't available as a separate package, we'll need to:

1. Use the core MLX framework
2. Implement our own tokenizer loading
3. Or use the `mlx_lm` Python bindings via a bridge (complex)

For now, let's start with just MLX core and implement model loading manually.

**Alternative**: If you find the exact LLM package URL, let me know and I'll update this.

## Step 4: Enable MLX in Code

After adding packages, uncomment the imports in `MLXEngine.swift`:

```swift
import MLX
import MLXNN
import MLXLLM
import MLXLMCommon
import Tokenizers
```

---

## Step 5: Build and Test

1. Select an **iOS Device** or **Apple Silicon Simulator** target
2. Build the project: `Cmd + B`
3. If successful, MLX is ready!

---

## Troubleshooting

### "No such module 'MLX'"

- Ensure packages were added correctly
- Clean build folder: `Cmd + Shift + K`
- Restart Xcode

### Build errors with MLX

- MLX requires iOS 16.0+ and Apple Silicon
- Check deployment target in Runner > General > Minimum Deployments

### Package resolution fails

- Check network connection
- Try: File > Packages > Reset Package Caches

---

## Verification

After setup, build and run. Check console for:

```
[MLXEngine] Initialized
[MLXEngine] mlxSwiftAvailable: true
```

If you see `mlxSwiftAvailable: true`, MLX is ready!

---

## Next Steps

Once MLX Swift is added:

1. Uncomment the MLX imports in `MLXEngine.swift`
2. Uncomment the actual model loading code
3. Test inference on device

## Related

^[source-materials/mirrors/doctrine/MLX_Swift_Setup_Instructions.md]
