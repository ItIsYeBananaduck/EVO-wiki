---
title: MLX_Swift_Integration_Plan
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/MLX_Swift_Integration_Plan.md
updated: 2026-07-24
---

# MLX Swift Integration Plan

**Goal**: Native MLX inference on iOS with dynamic LoRA support

---

## Current State

- ✅ MLX model on R2 (`alice-phi3-mlx-base-q4/model.safetensors`)
- ✅ `MLXEngine.swift` placeholder exists
- ✅ Fallback to llama.cpp works
- ❌ No actual MLX inference on iOS yet

---

## MLX Swift Capabilities

| Feature               | Status                      |
| --------------------- | --------------------------- |
| iOS/macOS support     | ✅ Yes                      |
| Swift Package Manager | ✅ Yes                      |
| LLM inference         | ✅ Yes (via MLXLLM)         |
| Safetensors loading   | ✅ Yes                      |
| LoRA adapters         | ✅ Yes (native safetensors) |
| Phi-3 models          | ✅ Yes                      |

---

## Integration Steps

### Phase 1: Add MLX Swift Package (30 min)

1. Add to `flutter_app/ios/Podfile` or use SPM:

```ruby
# Option A: CocoaPods (if available)
pod 'MLX', :git => 'https://github.com/ml-explore/mlx-swift.git'

# Option B: Swift Package Manager (preferred)
# Add in Xcode: File > Add Package Dependencies
# URL: https://github.com/ml-explore/mlx-swift.git
# Products: MLX, MLXNN, MLXRandom, MLXLLM
```

### Phase 2: Implement MLXEngine.swift (2-4 hours)

Replace placeholder with actual implementation:

```swift
import MLX
import MLXLLM
import Tokenizers

@objc class MLXEngine: NSObject {

    private var model: LLMModel?
    private var tokenizer: Tokenizer?
    private var loadedLoRAs: [String: LoRAAdapter] = [:]

    @objc func loadModel() -> Bool {
        guard let path = getModelPath() else { return false }

        do {
            // Load model configuration
            let config = try ModelConfiguration.load(from: URL(fileURLWithPath: path))

            // Load base model
            model = try LLMModel.load(configuration: config)

            // Load tokenizer
            tokenizer = try Tokenizer.load(from: URL(fileURLWithPath: path))

            _isLoaded = true
            return true
        } catch {
            _lastLoadError = error.localizedDescription
            return false
        }
    }

    @objc func loadLoRAAdapter(path: String, scale: Float, kind: String) -> Bool {
        guard let model = model else { return false }

        do {
            // MLX Swift can load safetensors LoRAs directly!
            let adapter = try LoRAAdapter.load(from: URL(fileURLWithPath: path))
            adapter.scale = scale
            loadedLoRAs[kind] = adapter

            // Apply to model
            model.applyLoRA(adapter)
            return true
        } catch {
            print("[MLXEngine] Failed to load LoRA: \(error)")
            return false
        }
    }

    @objc func generate(
        prompt: String,
        maxTokens: Int32,
        completion: @escaping (String?, Error?) -> Void
    ) {
        guard let model = model, let tokenizer = tokenizer else {
            completion(nil, NSError(domain: "MLXEngine", code: -1,
                userInfo: [NSLocalizedDescriptionKey: "Model not loaded"]))
            return
        }

        DispatchQueue.global(qos: .userInitiated).async {
            do {
                // Tokenize
                let inputIds = tokenizer.encode(prompt)

                // Generate
                let output = try model.generate(
                    inputIds: inputIds,
                    maxNewTokens: Int(maxTokens),
                    temperature: 0.7
                )

                // Decode
                let response = tokenizer.decode(output)

                DispatchQueue.main.async {
                    completion(response, nil)
                }
            } catch {
                DispatchQueue.main.async {
                    completion(nil, error)
                }
            }
        }
    }
}
```

### Phase 3: Model Format Compatibility (1 hour)

Verify our model works with MLXLLM:

1. Check `config.json` has correct architecture
2. Ensure tokenizer format is compatible
3. Test loading on macOS first (easier debugging)

### Phase 4: LoRA Adapter Integration (2 hours)

1. Confirm MLXLLM supports runtime LoRA loading
2. Test with U adapter (user personalization)
3. Implement adapter stacking (U + GU blend)

---

## Dependencies

Add to `Package.swift` or via Xcode:

```swift
dependencies: [
    .package(url: "https://github.com/ml-explore/mlx-swift", from: "0.21.0"),
    .package(url: "https://github.com/ml-explore/mlx-swift-examples", from: "1.0.0"),
]

// Products needed:
// - MLX (core)
// - MLXNN (neural network layers)
// - MLXLLM (language models)
// - MLXLMCommon (shared utilities)
```

---

## Testing Strategy

1. **macOS First**: Test MLX loading/inference on Mac (easier to debug)
2. **iOS Simulator**: Should work on Apple Silicon Macs
3. **iOS Device**: Final validation on real hardware

---

## Risks & Mitigations

| Risk                           | Mitigation                                 |
| ------------------------------ | ------------------------------------------ |
| Model format incompatible      | Use `mlx_lm.convert` to ensure format      |
| LoRA not supported in Swift    | Fall back to pre-merged adapters           |
| Performance issues             | Profile and optimize, compare to llama.cpp |
| Package conflicts with Flutter | Use separate Swift framework if needed     |

---

## Timeline

| Phase                 | Effort  | Status  |
| --------------------- | ------- | ------- |
| Add MLX Swift package | 30 min  | Pending |
| Implement MLXEngine   | 2-4 hrs | Pending |
| Model compatibility   | 1 hr    | Pending |
| LoRA integration      | 2 hrs   | Pending |
| Testing               | 2 hrs   | Pending |

**Total: ~1 day of focused work**

---

## Benefits Over llama.cpp

1. ✅ **Native safetensors** - No GGUF conversion needed
2. ✅ **Dynamic LoRA** - Load/unload adapters at runtime
3. ✅ **Apple optimized** - Metal acceleration out of the box
4. ✅ **Unified format** - Same model/adapters for training and inference
5. ✅ **Simpler pipeline** - No conversion step in nightly updates

## Related

^[source-materials/mirrors/doctrine/MLX_Swift_Integration_Plan.md]
