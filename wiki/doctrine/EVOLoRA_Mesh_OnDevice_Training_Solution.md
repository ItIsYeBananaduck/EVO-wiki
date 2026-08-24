---
title: EVOLoRA_Mesh_OnDevice_Training_Solution
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVOLoRA_Mesh_OnDevice_Training_Solution.md
updated: 2026-07-24
---

# EVOLoRA Mesh On-Device Training Solution

**Critical Requirement**: U (User) and T (Trainer) adapters must update **nightly on-device**.

**Challenge**: llama.cpp only supports GGUF format, but on-device conversion is too heavy.

---

## Architecture Overview

### Adapter Update Frequencies

| Adapter                 | Update Frequency      | Location | Format           |
| ----------------------- | --------------------- | -------- | ---------------- |
| **U** (User)            | **Nightly on-device** | Device   | ❓ Need solution |
| **T** (Trainer)         | **Nightly on-device** | Device   | ❓ Need solution |
| **GU** (Global User)    | Weekly (server)       | Server   | GGUF ✅          |
| **GT** (Global Trainer) | Weekly (server)       | Server   | GGUF ✅          |

---

## Solution Options

### Option 1: Lightweight On-Device GGUF Conversion (Recommended)

**Approach**: Use a minimal Python runtime + conversion script on-device.

**Pros**:

- ✅ Keeps all adapters in GGUF format
- ✅ Works with existing llama.cpp integration
- ✅ U/T adapters update nightly as required

**Cons**:

- ⚠️ Requires Python runtime on device (adds ~50-100MB)
- ⚠️ Conversion takes 1-2 minutes (but runs at night)
- ⚠️ Battery usage during conversion

**Implementation**:

```dart
// Flutter: Train U adapter nightly
Future<void> trainUserAdapter() async {
  // 1. Train LoRA adapter (safetensors)
  await qloraTrainer.runDailyTraining();
  // Output: adapter_model.safetensors

  // 2. Convert to GGUF (on-device, nightly)
  await convertLoraToGguf(
    loraDir: '/path/to/user_lora',
    outputPath: '/path/to/user_lora.gguf',
  );

  // 3. Load in llama.cpp
  await llamaEngine.loadLoRAAdapter(
    path: '/path/to/user_lora.gguf',
    scale: 1.0,
    kind: 'U',
  );
}
```

**Requirements**:

- Python runtime (via `python3` command or embedded)
- `convert_lora_to_gguf.py` script
- Minimal dependencies (transformers, torch, gguf)

---

### Option 2: Server-Side Conversion with Encrypted Upload

**Approach**: Train on-device, upload encrypted, server converts, download GGUF.

**Flow**:

```
Nightly:
1. Device: Train U adapter (safetensors)
2. Device: Encrypt and upload to server
3. Server: Decrypt, convert to GGUF
4. Server: Upload GGUF to R2
5. Device: Download GGUF (next check)
```

**Pros**:

- ✅ No on-device conversion overhead
- ✅ Server has full compute power
- ✅ All adapters in GGUF format

**Cons**:

- ❌ **Latency**: Update takes 2 steps (upload → download)
- ❌ **Privacy**: User data leaves device (even encrypted)
- ❌ **Complexity**: Two-step process

**Verdict**: ⚠️ **Not ideal** - adds latency and privacy concerns

---

### Option 3: Hybrid Inference Engine

**Approach**: Use llama.cpp for GU/GT (GGUF), different engine for U/T (safetensors).

**Options**:

- **MLX** (Apple Silicon): Native safetensors support
- **CoreML**: Convert safetensors to CoreML
- **ONNX Runtime**: Limited LoRA support

**Pros**:

- ✅ U/T adapters stay in safetensors (no conversion)
- ✅ Can update nightly without conversion

**Cons**:

- ❌ **Major architecture change** (lose llama.cpp optimizations)
- ❌ Two inference engines (complexity)
- ❌ Performance may be worse
- ❌ Adapter blending becomes complex (different engines)

**Verdict**: ❌ **Too disruptive** - not worth the complexity

---

### Option 4: Pre-Convert Template + Incremental Updates

**Approach**: Pre-convert base U/T adapter template to GGUF, then apply incremental updates.

**Flow**:

```
Initial:
1. Server: Pre-convert blank U/T adapter template to GGUF
2. Device: Download template GGUF

Nightly:
1. Device: Train LoRA delta (safetensors, small)
2. Device: Merge delta into template GGUF (lightweight operation)
3. Device: Load updated GGUF
```

**Pros**:

- ✅ Avoids full conversion (only merge delta)
- ✅ All adapters in GGUF format

**Cons**:

- ❌ **Complex**: GGUF doesn't support incremental updates natively
- ❌ Need custom merge logic
- ❌ Risk of corruption

**Verdict**: ⚠️ **Complex** - may not be feasible

---

## Recommended Solution: Option 1 (Lightweight On-Device Conversion)

### Implementation Plan

#### Step 1: Add Python Runtime to Flutter App

**Option A: Embedded Python (Recommended)**

- Use `dart_python` or similar package
- Bundle minimal Python runtime (~50MB)
- Include only required packages

**Option B: System Python**

- Require Python 3.9+ installed
- Use `Process.run()` to call Python
- Less reliable (user may not have Python)

#### Step 2: Bundle Conversion Script

**File**: `flutter_app/assets/scripts/convert_lora_to_gguf.py`

```python
#!/usr/bin/env python3
"""Lightweight LoRA to GGUF converter for on-device use."""
import sys
import json
from pathlib import Path

# Minimal imports (only what's needed)
try:
    import gguf
    import torch
    from transformers import AutoConfig
except ImportError as e:
    print(f"ERROR: Missing dependency: {e}", file=sys.stderr)
    sys.exit(1)

def convert(lora_dir: str, base_model: str, output_path: str):
    """Convert PEFT LoRA adapter to GGUF."""
    # Use existing convert_lora_to_gguf.py logic
    # But optimized for mobile (minimal memory)
    pass

if __name__ == "__main__":
    if len(sys.argv) != 4:
        print("Usage: convert_lora_to_gguf.py <lora_dir> <base_model> <output>")
        sys.exit(1)

    convert(sys.argv[1], sys.argv[2], sys.argv[3])
```

#### Step 3: Add Conversion to Existing Training Pipeline

**File**: `app/src/lib/services/ml/qloraTrainer.ts` (MODIFY)

```typescript
async runDailyTraining(): Promise<TrainingSession> {
  // ... existing training code ...

  // After training completes (line ~134):
  if (result.checkpointPath) {
    await this.persistTrainingOutputs(result.checkpointPath, signals.length);

    // NEW: Convert safetensors checkpoint to GGUF
    try {
      const ggufPath = await this.convertCheckpointToGguf(result.checkpointPath);

      // Store GGUF path for llama.cpp loading
      await this.storeGgufAdapter(ggufPath, 'U'); // User adapter

      console.log(`[QLoRATrainer] Converted adapter to GGUF: ${ggufPath}`);
    } catch (error) {
      console.error('[QLoRATrainer] GGUF conversion failed:', error);
      // Don't fail training if conversion fails - can retry later
    }
  }

  return session;
}

private async convertCheckpointToGguf(checkpointPath: string): Promise<string> {
  // Call Python conversion script
  const { exec } = await import('child_process');
  const { promisify } = await import('util');
  const execAsync = promisify(exec);

  const baseModel = 'microsoft/Phi-3-mini-4k-instruct';
  const outputPath = checkpointPath.replace('.safetensors', '.gguf');
  const scriptPath = 'assets/scripts/convert_lora_to_gguf.py';

  const command = `python3 ${scriptPath} ${checkpointPath} --base-model-id ${baseModel} --outfile ${outputPath}`;

  const { stdout, stderr } = await execAsync(command, {
    timeout: 120000, // 2 minute timeout
  });

  if (stderr && !stderr.includes('warning')) {
    throw new Error(`Conversion failed: ${stderr}`);
  }

  return outputPath;
}

private async storeGgufAdapter(ggufPath: string, adapterType: 'U' | 'T'): Promise<void> {
  // Store path in preferences for llama.cpp to load
  const { Preferences } = await import('@capacitor/preferences');
  await Preferences.set({
    key: `lora_adapter_${adapterType}_gguf_path`,
    value: ggufPath,
  });
}
```

#### Step 4: Load GGUF Adapter After Conversion

**File**: `app/src/lib/services/ml/modelUpdater.ts` (MODIFY)

```typescript
async loadUserLoRAAdapter(): Promise<void> {
  // Get GGUF path from preferences (set by qloraTrainer after conversion)
  const { Preferences } = await import('@capacitor/preferences');
  const ggufPath = await Preferences.get({ key: 'lora_adapter_U_gguf_path' });

  if (!ggufPath.value) {
    console.log('[ModelUpdater] No user LoRA adapter GGUF found');
    return;
  }

  // Load in llama.cpp via native plugin
  const { LlamaPlugin } = await import('./llamaPlugin');
  await LlamaPlugin.loadLoRAAdapter({
    path: ggufPath.value,
    scale: 1.0,
    kind: 'U',
  });

  console.log(`[ModelUpdater] Loaded user LoRA adapter: ${ggufPath.value}`);
}
```

#### Step 5: Background Task Integration (Already Exists)

**File**: `app/src/lib/services/ml/backgroundModelUpdater.ts` (MODIFY)

The nightly training is already triggered. We just need to ensure GGUF conversion happens and adapters are loaded:

```typescript
private async performNightlyCheck(trigger: string): Promise<void> {
  // ... existing model update code ...

  // Existing: Train user adapter nightly (already implemented)
  try {
    const { qloraTrainer } = await import('./qloraTrainer');
    await qloraTrainer.runDailyTraining();
    // Conversion happens inside runDailyTraining() now

    // Load the newly converted GGUF adapter
    await modelUpdater.loadUserLoRAAdapter();

    console.log('Nightly user adapter training and loading completed');
  } catch (error) {
    console.error('Error during user adapter training:', error);
  }
}
```

---

## Performance Considerations

### On-Device Conversion Time

**Estimated** (on modern iPhone):

- **Training**: 5-10 minutes (depends on data size)
- **Conversion**: 1-2 minutes
- **Total**: ~6-12 minutes nightly

**Optimization**:

- Run during background task (3 AM, user sleeping)
- Use low-power mode if available
- Skip conversion if adapter hasn't changed significantly

### Memory Usage

- **Training**: ~2-4GB RAM (QLoRA with 4-bit)
- **Conversion**: ~1-2GB RAM (temporary)
- **Total**: Manageable on modern devices

### Battery Impact

- **Training + Conversion**: ~5-10% battery (if plugged in, minimal impact)
- **Recommendation**: Only run when charging or battery > 50%

---

## Alternative: Simplified Conversion

If full conversion is too heavy, consider:

1. **Pre-quantized conversion**: Convert on server once, device applies quantized updates
2. **Delta compression**: Only convert changed weights
3. **Lazy conversion**: Convert on-demand (first use), cache result

---

## Implementation Checklist

- [ ] Add Python runtime to Flutter app (embedded or system)
- [ ] Bundle `convert_lora_to_gguf.py` script
- [ ] Create `OnDeviceLoRATrainer` class
- [ ] Implement on-device training pipeline
- [ ] Add conversion step after training
- [ ] Integrate with nightly background task
- [ ] Add battery/charging checks
- [ ] Test on real device (not just simulator)
- [ ] Optimize conversion for mobile (memory, speed)

---

## Summary

**Solution**: Lightweight on-device GGUF conversion

- ✅ U/T adapters train nightly on-device (as required) - **Already implemented** (`qloraTrainer.ts`)
- ✅ Convert to GGUF on-device (1-2 min, acceptable for nightly task) - **Need to add**
- ✅ All adapters in GGUF format (works with llama.cpp)
- ✅ Runs during background task (minimal user impact)

**Key Trade-off**: Acceptable conversion overhead (1-2 min nightly) to maintain architecture consistency and meet the nightly update requirement.

**Existing Infrastructure**:

- ✅ `qloraTrainer.ts` - Already trains LoRA adapters on-device
- ✅ `backgroundModelUpdater.ts` - Already runs nightly at 3 AM
- ✅ Training produces safetensors format
- ❌ **Missing**: Conversion step (safetensors → GGUF)

**What's Needed**:

1. Add conversion step after training in `qloraTrainer.ts`
2. Bundle `convert_lora_to_gguf.py` script
3. Add Python runtime (or use system Python)
4. Update adapter loading to use GGUF instead of safetensors

## Related

^[source-materials/mirrors/doctrine/EVOLoRA_Mesh_OnDevice_Training_Solution.md]
