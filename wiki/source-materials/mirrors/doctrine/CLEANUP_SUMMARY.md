---
title: CLEANUP_SUMMARY
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CLEANUP_SUMMARY.md"]
updated: 2026-07-24
---

# Codebase Cleanup Summary

## ✅ Completed Actions

### 1. Updated `.gitignore` Files

**Root `.gitignore`**:

- ✅ Enhanced `llama.cpp` build artifact ignores (build directories, model files)
- ✅ Added comprehensive `training/enf_lora/` ignores (outputs, checkpoints, downloads)
- ✅ Added exception for small ONNX files needed by the app (e.g., `movenet_singlepose_lightning.onnx`)

**New `training/enf_lora/.gitignore`**:

- ✅ Created dedicated `.gitignore` for training directory
- ✅ Ignores all training outputs, checkpoints, and large data files
- ✅ Keeps structure files (reports, scripts) but ignores large JSON/JSONL data

### 2. Verified Large Files Are Ignored

**Training Files**:

- ✅ `training/enf_lora/adapter_model.safetensors` - **Ignored** (not tracked)
- ✅ `training/enf_lora/output/gguf/*.gguf` - **Ignored** (all converted adapters)
- ✅ Training checkpoints and logs - **Ignored**

**Build Artifacts**:

- ✅ `alicecore_build/llama.cpp/build*/` - **Ignored** (submodule build artifacts)
- ✅ Model files in `alicecore_build/llama.cpp/models/*.gguf` - **Ignored**

### 3. Repository Status

**Current Repository Size**:

- Pack size: **6.50 GiB** (includes history)
- Loose objects: **442.56 MiB**
- Total tracked files: **2,819**

**Large Files Currently Tracked** (acceptable):

- `app/static/assets/Character_output-opt.glb` (6.0M) - 3D model asset (needed)
- `app/ios/App/App/Assets.xcassets/AppIcon.appiconset/AppIcon-1024x1024.png` (1.6M) - App icon (needed)
- `app/static/alice-animated.svg` (1.5M) - Animation asset (needed)

**Note**: These are legitimate assets needed by the app and are within acceptable size limits.

## 📋 Ignored File Patterns

### Model Files (All Ignored)

- `*.gguf` - GGUF model files
- `*.safetensors` - SafeTensors model files
- `*.onnx` - ONNX model files (except small ones like movenet)
- `*.pt`, `*.pth`, `*.ckpt` - PyTorch checkpoints
- `*.bin` - Binary model files

### Training Outputs (All Ignored)

- `training/enf_lora/output/` - All converted GGUF adapters
- `training/enf_lora/adapter_model.safetensors` - Source LoRA adapters
- `training/enf_lora/checkpoint-*/` - Training checkpoints
- `training/enf_lora/*.log` - Training logs
- `training/enf_lora/data/*.jsonl` - Large training data files

### Build Artifacts (All Ignored)

- `alicecore_build/llama.cpp/build*/` - All build directories
- `alicecore_build/llama.cpp/models/*.gguf` - Model vocab files
- `build/` - General build directories

## 🎯 Key Points

1. **Submodule Note**: `alicecore_build/llama.cpp` is a git submodule. Build artifacts inside it are managed by the submodule's own `.gitignore` and won't affect the main repository.

2. **Training Files**: All ENF/VOICE LoRA training outputs are properly ignored. The converted GGUF files (uploaded to R2) are not tracked.

3. **Asset Files**: Large asset files (3D models, icons) that are needed by the app are tracked, but they're within reasonable limits (< 10MB each).

4. **Model Files**: All model files (base models, adapters) are ignored. They're stored in R2 and downloaded by the app at runtime.

## ✅ Verification

Run these commands to verify:

```bash
# Check if large training files are ignored
git check-ignore -v training/enf_lora/output/gguf/*.gguf

# Check repository size
git count-objects -vH

# Find any large tracked files
git ls-files | xargs -I {} sh -c 'test -f {} && du -h {}' 2>/dev/null | awk '$1 ~ /[0-9]+[MG]/'
```

## 📝 Next Steps

1. ✅ All large files are properly ignored
2. ✅ Repository is clean and ready for commits
3. ✅ Training outputs won't accidentally be committed
4. ✅ Build artifacts are excluded

**The codebase is now clean and ready for git operations!**

## Related
