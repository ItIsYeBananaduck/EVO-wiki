---
title: EVOLoRA_Mesh_Runtime_Evaluation
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVOLoRA_Mesh_Runtime_Evaluation.md
updated: 2026-07-24
---

# EVOLoRA Mesh Runtime Evaluation

**Problem**: llama.cpp only supports GGUF, requiring nightly safetensors→GGUF conversion which doesn't scale.

**Question**: Should we switch to a runtime that natively supports safetensors?

---

## Runtime Options

### Option 1: MLX (Apple Silicon Native)

**Pros**:

- ✅ **Native safetensors support** - No conversion needed
- ✅ **Optimized for Apple Silicon** - M-series chips
- ✅ **LoRA support** - Can load/apply LoRA adapters dynamically
- ✅ **Incremental training** - Can continue training from previous adapter
- ✅ **Fast inference** - Metal acceleration
- ✅ **Active development** - Apple-backed, actively maintained

**Cons**:

- ❌ **iOS deployment complexity** - Need to build XCFramework
- ❌ **Flutter integration** - Need to create plugin/bindings
- ❌ **Model format** - May need to convert base model to MLX format
- ❌ **Less mature** - Smaller community than llama.cpp

**LoRA Support**:

```python
# MLX supports LoRA natively
import mlx.core as mx
from mlx_lm import load, generate

# Load base model
model, tokenizer = load("microsoft/Phi-3-mini-4k-instruct")

# Load LoRA adapter (safetensors)
from mlx_lm.lora import LoRALinear
model = load_lora_weights(model, "user_lora.safetensors")

# Apply multiple adapters with scales
model = apply_lora_stack(model, [
    {"path": "user_lora.safetensors", "scale": 0.7},
    {"path": "global_user_lora.safetensors", "scale": 0.3},
])
```

**Incremental Training**:

```python
# MLX supports continuing from previous adapter
from mlx_lm.tuner import train

# Load previous adapter
previous_adapter = load_lora_weights(model, "user_lora.safetensors")

# Continue training
train(
    model=model,
    adapter_path="user_lora.safetensors",  # Updates existing
    train_data=new_data,
)
```

**Verdict**: ✅ **Best option** - Native safetensors, incremental training, Apple-optimized

---

### Option 2: CoreML (Apple's ML Framework)

**Pros**:

- ✅ **Native iOS** - Built into iOS
- ✅ **Hardware acceleration** - Neural Engine + GPU
- ✅ **No conversion** - Can convert models to CoreML format

**Cons**:

- ❌ **LoRA support unclear** - May not support dynamic LoRA loading
- ❌ **Limited flexibility** - Harder to apply multiple adapters
- ❌ **Training support** - CoreML is primarily for inference
- ❌ **Model conversion** - Need to convert base model to CoreML

**Verdict**: ⚠️ **Unclear** - Need to verify LoRA support

---

### Option 3: ONNX Runtime

**Pros**:

- ✅ **Cross-platform** - iOS, Android, Web
- ✅ **LoRA support** - Some support via extensions
- ✅ **Optimized** - Good performance

**Cons**:

- ❌ **Limited LoRA support** - Not as flexible as MLX
- ❌ **Complex setup** - ONNX model conversion
- ❌ **Incremental training** - May not support continuing from previous adapter

**Verdict**: ⚠️ **Limited** - LoRA support is not as robust

---

### Option 4: Transformers.js (Web-based)

**Pros**:

- ✅ **Native safetensors** - Supports HuggingFace format
- ✅ **LoRA support** - Can load LoRA adapters

**Cons**:

- ❌ **Web-only** - Not native iOS
- ❌ **Performance** - Slower than native
- ❌ **Not suitable** - For mobile app

**Verdict**: ❌ **Not applicable** - Web-only

---

### Option 5: Keep llama.cpp + Optimize Conversion

**Approach**: Accept conversion overhead, but optimize it

**Optimizations**:

- Lazy conversion (only when adapter changes)
- Incremental conversion (only convert changed weights)
- Background conversion (lower priority)
- Cache converted adapters

**Pros**:

- ✅ Keep existing llama.cpp integration
- ✅ Proven performance
- ✅ Large community

**Cons**:

- ❌ Still requires conversion (overhead)
- ❌ Doesn't solve scaling issue
- ❌ Battery/storage overhead

**Verdict**: ⚠️ **Workaround, not solution** - Doesn't address root cause

---

## Recommendation: MLX

### Why MLX?

1. **Native safetensors** - No conversion needed
2. **Incremental training** - Can continue from previous adapter
3. **Apple-optimized** - Built for M-series chips
4. **Dynamic LoRA** - Can load/apply adapters at runtime
5. **Active development** - Apple-backed, growing ecosystem
6. **Already in use** - You already have `train_phi3_mlx.py` in your codebase!

**Existing Infrastructure**:

- ✅ `training/train_phi3_mlx.py` - Already using MLX for training
- ✅ `training/alice-phi3-mlx/adapters/adapters.safetensors` - MLX produces safetensors natively
- ✅ MLX training pipeline is established
- ✅ MLX LoRA adapters already being trained

**What's Missing**:

- ❌ MLX inference runtime for iOS (need to build XCFramework or use Python runtime)
- ❌ MLX Capacitor plugin for Flutter
- ❌ Integration with existing inference pipeline (currently using llama.cpp)

**Key Insight**: You're already training with MLX and producing safetensors. The logical next step is to use MLX for inference too, eliminating the conversion step entirely.

### Migration Path

#### Phase 1: Evaluate MLX (1-2 weeks)

- [ ] Test MLX inference performance vs llama.cpp
- [ ] Verify LoRA loading/application works
- [ ] Test incremental training capability
- [ ] Measure memory usage and battery impact
- [ ] Build XCFramework for iOS

#### Phase 2: Create MLX Plugin (2-3 weeks)

- [ ] Create Capacitor plugin for MLX
- [ ] Implement inference API (match llama.cpp interface)
- [ ] Implement LoRA loading API
- [ ] Implement incremental training API
- [ ] Test on real device

#### Phase 3: Migrate Inference (1-2 weeks)

- [ ] Replace llama.cpp calls with MLX calls
- [ ] Update `LlamaEngine.swift` → `MLXEngine.swift`
- [ ] Update Flutter integration
- [ ] Test end-to-end

#### Phase 4: Update Training (1 week)

- [ ] Migrate training to use MLX
- [ ] Update `qloraTrainer.ts` to use MLX
- [ ] Test incremental training

**Total**: ~6-8 weeks for full migration

---

## MLX Implementation Preview

### Swift Plugin

**File**: `flutter_app/ios/Runner/MLXEngine.swift` (NEW)

```swift
import MLX
import MLXNN
import MLXOptimizers

@objc class MLXEngine: NSObject {
    @objc static let shared = MLXEngine()

    private var model: LanguageModel?
    private var loadedAdapters: [LoRAAdapter] = []

    /// Load base model
    @objc func loadModel(path: String) -> Bool {
        // Load MLX model (converted from HuggingFace)
        model = try? LanguageModel.load(from: path)
        return model != nil
    }

    /// Load LoRA adapter (safetensors - no conversion!)
    @objc func loadLoRAAdapter(
        path: String,
        scale: Float,
        kind: String
    ) -> Bool {
        guard let model = model else { return false }

        // Load safetensors directly (no conversion!)
        let adapter = try? LoRAAdapter.load(from: path)
        guard let adapter = adapter else { return false }

        // Apply adapter with scale
        model.applyLoRA(adapter, scale: scale)

        loadedAdapters.append(LoRAAdapter(
            adapter: adapter,
            path: path,
            scale: scale,
            kind: kind
        ))

        return true
    }

    /// Continue training from previous adapter
    @objc func continueTraining(
        previousAdapterPath: String?,
        newData: [[String: Any]],
        epochs: Int32
    ) -> Bool {
        // Load previous adapter if exists
        if let prevPath = previousAdapterPath {
            let prevAdapter = try? LoRAAdapter.load(from: prevPath)
            model?.loadLoRAWeights(prevAdapter)
        }

        // Continue training with new data
        // ... training loop ...

        // Save updated adapter (safetensors)
        let updatedAdapter = model?.extractLoRAWeights()
        try? updatedAdapter?.save(to: previousAdapterPath ?? "/path/to/new_adapter.safetensors")

        return true
    }

    /// Generate text
    @objc func generate(
        prompt: String,
        maxTokens: Int32
    ) -> String? {
        guard let model = model else { return nil }
        return model.generate(prompt: prompt, maxTokens: Int(maxTokens))
    }
}
```

### Flutter Integration

**File**: `flutter_app/lib/features/alice/domain/mlx_brain_service.dart` (NEW)

```dart
class MLXBrainService implements AliceBrainService {
  final MethodChannel _channel = MethodChannel('evo/mlx_engine');

  @override
  Future<AliceBrainResponse> generate(AliceBrainRequest request) async {
    // Get adapter stack from MeshRouter
    final adapterStack = await meshRouter.getAdapterStackForNative(
      user: request.context.user,
      domain: request.context.domain.name,
    );

    // Load adapters (safetensors - no conversion!)
    for (final adapter in adapterStack) {
      await _channel.invokeMethod('loadLoRAAdapter', {
        'path': adapter['path'],
        'scale': adapter['scale'],
        'kind': adapter['kind'],
      });
    }

    // Generate
    final result = await _channel.invokeMethod('generate', {
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

## Comparison Matrix

| Feature                     | llama.cpp      | MLX              | CoreML | ONNX |
| --------------------------- | -------------- | ---------------- | ------ | ---- |
| **Safetensors native**      | ❌             | ✅               | ⚠️     | ⚠️   |
| **LoRA support**            | ✅ (GGUF only) | ✅ (safetensors) | ❓     | ⚠️   |
| **Incremental training**    | ❌             | ✅               | ❌     | ⚠️   |
| **Dynamic adapter loading** | ✅             | ✅               | ❓     | ⚠️   |
| **Apple Silicon optimized** | ✅             | ✅✅             | ✅✅   | ⚠️   |
| **iOS deployment**          | ✅             | ⚠️               | ✅     | ⚠️   |
| **Community/ecosystem**     | ✅✅           | ✅               | ✅     | ✅   |
| **Conversion overhead**     | ❌ (required)  | ✅ (none)        | ⚠️     | ⚠️   |

---

## Decision Framework

### If you prioritize:

- **No conversion overhead** → MLX
- **Proven stability** → llama.cpp (with conversion)
- **Native iOS integration** → CoreML (if LoRA supported)
- **Cross-platform** → ONNX

### If you need:

- **Incremental training** → MLX (only option that truly supports it)
- **Dynamic adapter switching** → MLX or llama.cpp
- **Best performance** → MLX (Apple Silicon) or llama.cpp (Metal)

---

## Next Steps

1. **Evaluate MLX** - Test if it meets requirements
2. **If MLX works** - Plan migration (6-8 weeks)
3. **If MLX doesn't work** - Optimize llama.cpp conversion (lazy, incremental)
4. **Hybrid approach** - Use MLX for U/T adapters, llama.cpp for GU/GT (if MLX deployment is complex)

---

## Hybrid Approach (Fallback)

If full MLX migration is too complex:

- **U/T adapters**: Use MLX (incremental training, no conversion)
- **GU/GT adapters**: Use llama.cpp (weekly server conversion, no on-device training)

This gives you:

- ✅ Incremental training for U/T (MLX)
- ✅ No conversion for U/T (MLX)
- ✅ Keep existing llama.cpp for GU/GT (proven, stable)
- ✅ Minimal migration risk (only U/T adapters)

---

## Recommendation

**Start with MLX evaluation** (1-2 weeks):

1. Test MLX inference performance
2. Verify LoRA support
3. Test incremental training
4. Build iOS XCFramework

**If MLX works well**:

- Migrate to MLX (6-8 weeks)
- Eliminate conversion overhead
- Enable true incremental training

**If MLX doesn't work**:

- Optimize llama.cpp conversion (lazy, incremental)
- Accept conversion overhead as trade-off

The conversion overhead (1-2 min nightly) may be acceptable if MLX migration is too complex. But if you need true incremental training without conversion, MLX is the best path forward.

## Related

^[source-materials/mirrors/doctrine/EVOLoRA_Mesh_Runtime_Evaluation.md]
