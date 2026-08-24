---
title: MLX_Swift_Integration_Status
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/MLX_Swift_Integration_Status.md
updated: 2026-07-24
---

# MLX Swift Integration Status

## ✅ Completed

1. **MLX Swift Package Added**
   - ✅ Cloned locally to `flutter_app/ios/Packages/mlx-swift/`
   - ✅ Added to Xcode project via local path
   - ✅ Products linked: MLX, MLXNN, MLXFast, MLXLinalg, MLXFFT
   - ✅ Code compiles successfully

2. **MLXEngine.swift Structure**
   - ✅ Imports MLX and MLXNN (conditional compilation)
   - ✅ Model path detection (App Group & Documents)
   - ✅ Model validation (checks for required files)
   - ✅ LoRA adapter registration framework
   - ✅ Diagnostic status reporting
   - ✅ Fallback chain integrated (AliceInferenceManager)

3. **Infrastructure**
   - ✅ MLX model download in AliceAssetDownloadManager
   - ✅ Update script: `flutter_app/ios/Packages/update-mlx-swift.sh`
   - ✅ Documentation updated

## 🔄 In Progress

### Model Loading Implementation

**Current Status**: Structure ready, but full Phi-3 model loading not yet implemented.

**Why**: Loading a Phi-3 transformer model requires:

1. Parsing `config.json` to get architecture parameters (hidden_size, num_layers, num_heads, etc.)
2. Loading `model.safetensors` and parsing weight tensors
3. Constructing Phi-3 architecture using MLXNN components:
   - Embedding layers
   - Transformer blocks (attention + MLP)
   - Output head
4. Loading tokenizer from `tokenizer.json`

**Options**:

#### Option A: Use mlx-swift-lm (If Phi-3 Supported)

```swift
// If mlx-swift-lm has Phi-3 support:
import MLXLLM
let model = try Phi3.load(from: modelURL)
```

**Check**: https://github.com/ml-explore/mlx-swift-lm

- Does it support Phi-3?
- What's the API?

#### Option B: Build Custom Phi-3 Loader

Implement using MLXNN components:

```swift
import MLXNN

// Parse config
struct Phi3Config: Codable {
    let hiddenSize: Int
    let numLayers: Int
    let numHeads: Int
    // ... etc
}

// Load weights from safetensors
let weights = try loadSafetensors(from: weightsURL)

// Construct model
class Phi3Model: Module {
    let embedding: Embedding
    let layers: [TransformerBlock]
    // ...
}

// Load weights into model
model.loadWeights(weights)
```

**Complexity**: Medium-High (requires understanding Phi-3 architecture)

## 📋 Next Steps

1. **Research mlx-swift-lm**
   - Check if Phi-3 is supported
   - If yes: Add package and use it
   - If no: Proceed with Option B

2. **Implement Model Loading** (if Option B)
   - Parse config.json
   - Load safetensors (MLX has utilities for this)
   - Build Phi-3 architecture using MLXNN.Transformer
   - Load weights into architecture

3. **Implement Tokenizer Loading**
   - Load tokenizer.json
   - Parse tokenizer config
   - Create encoding/decoding functions

4. **Implement Generation**
   - Tokenize input
   - Forward pass through model
   - Sample tokens
   - Decode output

5. **Implement LoRA Loading**
   - Parse LoRA safetensors
   - Extract delta_A/delta_B matrices
   - Apply to model layers with scale

6. **Test**
   - Test on iOS simulator first
   - Test on real device
   - Benchmark performance vs llama.cpp

## 🚀 Current Fallback

Until MLX loading is complete:

- ✅ MLXEngine returns helpful error messages
- ✅ AliceInferenceManager falls back to llama.cpp
- ✅ App continues to work normally

## 📝 Notes

- MLX Swift supports safetensors natively (no conversion needed!)
- MLX uses Metal for GPU acceleration on Apple Silicon
- Should provide better performance than llama.cpp on iOS
- LoRA adapters can be loaded directly (no GGUF conversion)

## 🔗 Resources

- MLX Swift Docs: https://swiftpackageindex.com/ml-explore/mlx-swift
- MLX Swift Examples: https://github.com/ml-explore/mlx-swift-examples
- MLX LLM Package: https://github.com/ml-explore/mlx-swift-lm
- MLX Python (reference): https://ml-explore.github.io/mlx

## Related

^[source-materials/mirrors/doctrine/MLX_Swift_Integration_Status.md]
