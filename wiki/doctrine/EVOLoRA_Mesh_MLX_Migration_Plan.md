---
title: EVOLoRA_Mesh_MLX_Migration_Plan
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVOLoRA_Mesh_MLX_Migration_Plan.md
updated: 2026-07-24
---

# EVOLoRA Mesh MLX Migration Plan

**Current State**: Training with MLX (safetensors), inference with llama.cpp (GGUF)
**Target State**: Both training and inference with MLX (safetensors throughout)

**Why**: Eliminate safetensors→GGUF conversion overhead, enable true incremental training

---

## Architecture Comparison

### Current (llama.cpp)

```
Training: MLX → safetensors
Inference: llama.cpp → GGUF (requires conversion)
Problem: Conversion overhead, can't continue training on GGUF
```

### Target (MLX)

```
Training: MLX → safetensors
Inference: MLX → safetensors (no conversion!)
Benefit: No conversion, incremental training works
```

---

## MLX iOS Deployment Options

### Option A: Python Runtime + MLX (Recommended)

**Approach**: Use Python runtime on iOS, run MLX inference via Python

**Pros**:

- ✅ MLX is Python-based (native support)
- ✅ Can reuse existing MLX training code
- ✅ No need to build C++ bindings
- ✅ Incremental training works natively

**Cons**:

- ⚠️ Need Python runtime on iOS (~50-100MB)
- ⚠️ Python interop overhead (but acceptable for inference)

**Implementation**:

```swift
// Swift: Call Python MLX script
@objc func generateWithMLX(prompt: String) -> String? {
    let script = """
        import mlx.core as mx
        from mlx_lm import load, generate

        model, tokenizer = load("path/to/model")
        # Load LoRA adapters (safetensors - no conversion!)
        model = load_lora_weights(model, "user_lora.safetensors")

        response = generate(model, tokenizer, prompt=prompt, max_tokens=256)
        print(response)
    """

    // Execute Python script
    return executePythonScript(script)
}
```

### Option B: MLX Swift Bindings

**Approach**: Build Swift bindings for MLX C++ core

**Pros**:

- ✅ Native Swift performance
- ✅ No Python overhead
- ✅ Better integration

**Cons**:

- ❌ Complex (need to build bindings)
- ❌ MLX may not have official Swift support
- ❌ Higher development effort

**Verdict**: ⚠️ **Complex** - May not be worth the effort

### Option C: Hybrid - MLX for U/T, llama.cpp for GU/GT

**Approach**: Use MLX only for user/trainer adapters (incremental), keep llama.cpp for global

**Pros**:

- ✅ Minimal migration (only U/T adapters)
- ✅ Keep proven llama.cpp for GU/GT
- ✅ Eliminates conversion for U/T (most critical)

**Cons**:

- ⚠️ Two inference engines (complexity)
- ⚠️ Adapter blending across engines is complex

**Verdict**: ⚠️ **Possible fallback** - If full MLX migration is too complex

---

## Recommended: Option A (Python Runtime + MLX)

### Implementation Plan

#### Step 1: Add Python Runtime to iOS

**Option 1: Embedded Python**

- Use `PythonKit` or similar
- Bundle Python 3.9+ (~50MB)
- Include MLX and dependencies

**Option 2: System Python** (if available)

- Use system Python if iOS allows
- Less reliable (user may not have it)

**Recommendation**: Embedded Python for reliability

#### Step 2: Create MLX Inference Script

**File**: `flutter_app/assets/scripts/mlx_inference.py` (NEW)

```python
#!/usr/bin/env python3
"""MLX inference for Alice - supports safetensors LoRA adapters natively."""
import sys
import json
from pathlib import Path

import mlx.core as mx
from mlx_lm import load, generate

def load_model_and_adapters(
    base_model_path: str,
    adapter_paths: list[dict]  # [{"path": "...", "scale": 0.7, "kind": "U"}, ...]
):
    """Load base model and apply LoRA adapters."""
    # Load base model
    model, tokenizer = load(base_model_path)

    # Load and apply adapters (safetensors - no conversion!)
    for adapter_config in adapter_paths:
        adapter_path = adapter_config["path"]
        scale = adapter_config.get("scale", 1.0)

        # Load LoRA adapter (safetensors)
        from mlx_lm.lora import LoRALinear
        adapter_weights = load_lora_weights(adapter_path)

        # Apply with scale
        model = apply_lora_with_scale(model, adapter_weights, scale)

    return model, tokenizer

def generate_response(
    model,
    tokenizer,
    prompt: str,
    max_tokens: int = 256
) -> str:
    """Generate response using MLX."""
    response = generate(
        model,
        tokenizer,
        prompt=prompt,
        max_tokens=max_tokens,
        temp=0.7,
    )
    return response

if __name__ == "__main__":
    # Parse arguments
    args = json.loads(sys.stdin.read())

    # Load model and adapters
    model, tokenizer = load_model_and_adapters(
        args["base_model_path"],
        args["adapter_paths"]
    )

    # Generate
    response = generate_response(
        model,
        tokenizer,
        args["prompt"],
        args.get("max_tokens", 256)
    )

    # Output JSON
    print(json.dumps({"text": response}))
```

#### Step 3: Create Swift MLX Engine

**File**: `flutter_app/ios/Runner/MLXEngine.swift` (NEW)

```swift
import Foundation
import PythonKit

@objc class MLXEngine: NSObject {
    @objc static let shared = MLXEngine()

    private var pythonInitialized = false

    private func ensurePythonInitialized() {
        guard !pythonInitialized else { return }

        // Initialize Python
        let python = Python.import("sys")
        let scriptPath = Bundle.main.path(forResource: "mlx_inference", ofType: "py", inDirectory: "scripts")
        if let scriptPath = scriptPath {
            python.path.insert(0, scriptPath)
        }

        pythonInitialized = true
    }

    @objc func generate(
        baseModelPath: String,
        adapterPaths: [[String: Any]],
        prompt: String,
        maxTokens: Int32 = 256
    ) -> String? {
        ensurePythonInitialized()

        let mlx = Python.import("mlx_inference")

        let args: [String: Any] = [
            "base_model_path": baseModelPath,
            "adapter_paths": adapterPaths,
            "prompt": prompt,
            "max_tokens": maxTokens,
        ]

        let argsJson = try? JSONSerialization.data(withJSONObject: args)
        let argsString = String(data: argsJson ?? Data(), encoding: .utf8) ?? "{}"

        // Call Python script
        let result = mlx.generate_response(argsString)

        if let resultDict = try? JSONSerialization.jsonObject(with: Data(result.utf8), options: []) as? [String: Any],
           let text = resultDict["text"] as? String {
            return text
        }

        return nil
    }

    @objc func continueTraining(
        previousAdapterPath: String?,
        newData: [[String: Any]],
        epochs: Int32
    ) -> Bool {
        ensurePythonInitialized()

        // Use existing MLX training code
        // Load previous adapter, continue training, save updated adapter
        // ... implementation ...

        return true
    }
}
```

#### Step 4: Update Flutter Integration

**File**: `flutter_app/lib/features/alice/domain/alice_brain_service.dart` (MODIFY)

```dart
class MethodChannelAliceBrainService implements AliceBrainService {
  // Add MLX engine option
  final bool useMLX = true; // Feature flag

  @override
  Future<AliceBrainResponse> generate(AliceBrainRequest request) async {
    // ... existing code ...

    final adapterStack = await _buildAdapterStack(request, autonomy);

    if (useMLX) {
      // Use MLX engine (safetensors, no conversion)
      return await _generateWithMLX(request, adapterStack);
    } else {
      // Fallback to llama.cpp (existing)
      return await _generateWithLlama(request, adapterStack);
    }
  }

  Future<AliceBrainResponse> _generateWithMLX(
    AliceBrainRequest request,
    List<Map<String, dynamic>> adapterStack,
  ) async {
    // Call MLX engine via method channel
    final result = await _channel.invokeMethod('generate_mlx', {
      'baseModelPath': '/path/to/base_model',
      'adapterPaths': adapterStack, // Safetensors paths - no conversion!
      'prompt': request.userMessage,
      'maxTokens': 256,
    });

    return AliceBrainResponse(
      text: result['text'],
      // ...
    );
  }
}
```

---

## Migration Timeline

### Phase 1: Evaluation (1 week)

- [ ] Test MLX inference performance vs llama.cpp
- [ ] Verify Python runtime on iOS works
- [ ] Test LoRA loading (safetensors)
- [ ] Measure memory/battery impact

### Phase 2: MLX Engine (2 weeks)

- [ ] Add Python runtime to iOS app
- [ ] Create MLX inference script
- [ ] Create Swift MLXEngine wrapper
- [ ] Test basic inference

### Phase 3: Integration (1 week)

- [ ] Update Flutter integration
- [ ] Add feature flag (MLX vs llama.cpp)
- [ ] Test end-to-end

### Phase 4: Training Integration (1 week)

- [ ] Update on-device training to use MLX
- [ ] Test incremental training
- [ ] Verify adapter persistence

**Total**: ~5 weeks

---

## Performance Expectations

### MLX vs llama.cpp

| Metric                   | llama.cpp         | MLX (Python)             | MLX (Native)         |
| ------------------------ | ----------------- | ------------------------ | -------------------- |
| **Inference speed**      | Fast (Metal)      | Medium (Python overhead) | Fast (Metal)         |
| **Memory usage**         | Low               | Medium                   | Low                  |
| **LoRA loading**         | GGUF (conversion) | Safetensors (native)     | Safetensors (native) |
| **Incremental training** | ❌                | ✅                       | ✅                   |
| **Conversion overhead**  | ❌ (required)     | ✅ (none)                | ✅ (none)            |

**Trade-off**: Accept Python overhead (~10-20% slower) to eliminate conversion and enable incremental training.

---

## Decision Point

**If MLX works well**:

- ✅ Migrate to MLX (5 weeks)
- ✅ Eliminate conversion overhead
- ✅ Enable incremental training
- ✅ Native safetensors support

**If MLX doesn't work**:

- ⚠️ Optimize llama.cpp conversion (lazy, incremental)
- ⚠️ Accept conversion overhead as trade-off
- ⚠️ Consider hybrid approach (MLX for U/T, llama.cpp for GU/GT)

---

## Next Steps

1. **Immediate**: Test MLX inference on macOS (proof of concept)
2. **Week 1**: Evaluate Python runtime on iOS
3. **Week 2-3**: Build MLX engine if evaluation succeeds
4. **Week 4-5**: Integrate and test

**Recommendation**: Start with MLX evaluation. If it works, migrate. If not, optimize llama.cpp conversion as fallback.

## Related

^[source-materials/mirrors/doctrine/EVOLoRA_Mesh_MLX_Migration_Plan.md]
