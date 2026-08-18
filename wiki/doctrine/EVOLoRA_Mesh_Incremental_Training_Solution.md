---
title: EVOLoRA_Mesh_Incremental_Training_Solution
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_Incremental_Training_Solution.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh Incremental Training Solution

**Problem**: GGUF format is read-only. If we convert to GGUF, we can't continue incremental training.

**Requirement**: U/T adapters must update **nightly** and **incrementally** (continue from previous adapter, not retrain from scratch).

---

## Solution: Dual-Format Approach

**Keep both formats**:

- **Safetensors**: For training (incremental, can continue from previous)
- **GGUF**: For inference (converted from safetensors, read-only)

**Flow**:

```
Nightly:
1. Load previous safetensors adapter (if exists)
2. Continue training incrementally (add new data)
3. Save updated safetensors
4. Convert safetensors → GGUF (for inference)
5. Load GGUF in llama.cpp
```

---

## Implementation

### Step 1: Modify Training to Support Incremental Updates

**File**: `app/src/lib/services/ml/qloraTrainer.ts` (MODIFY)

```typescript
async runDailyTraining(): Promise<TrainingSession> {
  // ... existing code to get signals ...

  // NEW: Load previous safetensors adapter if exists (for incremental training)
  const previousAdapterPath = await this.getPreviousAdapterPath('U');

  if (previousAdapterPath) {
    console.log(`[QLoRATrainer] Continuing from previous adapter: ${previousAdapterPath}`);
  } else {
    console.log('[QLoRATrainer] No previous adapter found - training from scratch');
  }

  // Run QLoRA training (incremental if previous adapter exists)
  const result = await LlamaPlugin.trainQLoRA({
    rank: QLORA_CONFIG.rank,
    epochs: QLORA_CONFIG.epochs,
    learningRate: QLORA_CONFIG.learningRate,
    signals: trainingData.inputs,
    expectedOutputs: trainingData.outputs,
    previousAdapterPath: previousAdapterPath || undefined, // NEW: Continue from previous
  });

  // ... existing code ...

  if (result.checkpointPath) {
    // 1. Save safetensors (for next night's incremental training)
    await this.saveAdapterForIncrementalTraining(
      result.checkpointPath,
      'U'
    );

    // 2. Convert to GGUF (for inference)
    const ggufPath = await this.convertCheckpointToGguf(result.checkpointPath);

    // 3. Load GGUF in llama.cpp
    await this.loadGgufAdapter(ggufPath, 'U');

    // 4. Keep safetensors for next night
    // (don't delete - needed for incremental training)
  }

  return session;
}

private async getPreviousAdapterPath(adapterType: 'U' | 'T'): Promise<string | null> {
  const { Preferences } = await import('@capacitor/preferences');
  const path = await Preferences.get({
    key: `lora_adapter_${adapterType}_safetensors_path`
  });
  return path.value || null;
}

private async saveAdapterForIncrementalTraining(
  checkpointPath: string,
  adapterType: 'U' | 'T'
): Promise<void> {
  // Copy checkpoint to permanent location for next night
  const permanentPath = `/path/to/adapters/${adapterType}_adapter.safetensors`;

  // Copy file
  const { Filesystem, Directory } = await import('$lib/adapters/filesystem');
  const checkpointData = await Filesystem.readFile({
    path: checkpointPath,
    directory: Directory.Data,
  });

  await Filesystem.writeFile({
    path: permanentPath,
    data: checkpointData.data,
    directory: Directory.Data,
  });

  // Store path for next night
  const { Preferences } = await import('@capacitor/preferences');
  await Preferences.set({
    key: `lora_adapter_${adapterType}_safetensors_path`,
    value: permanentPath,
  });
}
```

### Step 2: Update Native Training to Support Incremental

**File**: `app/src/lib/services/ml/llamaPlugin.ts` (MODIFY)

```typescript
export interface LlamaPluginInterface {
  trainQLoRA(options: {
    rank: number;
    epochs: number;
    learningRate: number;
    signals: TrainingInput["signals"];
    expectedOutputs: number[];
    previousAdapterPath?: string; // NEW: Path to previous safetensors adapter
  }): Promise<TrainingResult>;
}
```

**File**: `flutter_app/ios/Runner/LlamaEngine.swift` (MODIFY - if native training exists)

**OR** if training happens in a different backend (Python/MLX), update that instead.

**Key Point**: The training backend must support loading a previous safetensors adapter and continuing training from those weights, not starting from scratch.

### Step 3: Conversion After Each Training Session

**File**: `app/src/lib/services/ml/qloraTrainer.ts` (MODIFY)

```typescript
private async convertCheckpointToGguf(checkpointPath: string): Promise<string> {
  // Convert safetensors → GGUF (for inference only)
  // This happens every night, but it's just conversion, not retraining

  const baseModel = 'microsoft/Phi-3-mini-4k-instruct';
  const outputPath = checkpointPath.replace('.safetensors', '.gguf');

  // Call Python conversion script
  const { exec } = await import('child_process');
  const { promisify } = await import('util');
  const execAsync = promisify(exec);

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

private async loadGgufAdapter(ggufPath: string, adapterType: 'U' | 'T'): Promise<void> {
  // Load GGUF in llama.cpp for inference
  const { LlamaPlugin } = await import('./llamaPlugin');
  await LlamaPlugin.loadLoRAAdapter({
    path: ggufPath,
    scale: 1.0,
    kind: adapterType,
  });

  // Store GGUF path (for inference)
  const { Preferences } = await import('@capacitor/preferences');
  await Preferences.set({
    key: `lora_adapter_${adapterType}_gguf_path`,
    value: ggufPath,
  });
}
```

---

## File Management

### Storage Structure

```
Device Storage:
├── adapters/
│   ├── U_adapter.safetensors      ← For training (incremental)
│   ├── U_adapter.gguf             ← For inference (converted)
│   ├── T_adapter.safetensors      ← For training (incremental)
│   └── T_adapter.gguf              ← For inference (converted)
```

### Lifecycle

**Night 1**:

1. Train from scratch → `U_adapter.safetensors`
2. Convert → `U_adapter.gguf`
3. Load GGUF for inference

**Night 2**:

1. Load `U_adapter.safetensors` (previous)
2. Continue training (incremental) → Update `U_adapter.safetensors`
3. Convert updated safetensors → `U_adapter.gguf` (overwrite)
4. Load new GGUF for inference

**Night 3+**: Same as Night 2 (incremental)

---

## Alternative: Lazy Conversion

If conversion is too slow for nightly, use **lazy conversion**:

```typescript
async runDailyTraining(): Promise<TrainingSession> {
  // ... training code ...

  // Save safetensors (always)
  await this.saveAdapterForIncrementalTraining(result.checkpointPath, 'U');

  // Convert to GGUF only if:
  // 1. No GGUF exists yet, OR
  // 2. Safetensors changed significantly (checksum different)
  const needsConversion = await this.shouldConvertToGguf('U');

  if (needsConversion) {
    const ggufPath = await this.convertCheckpointToGguf(result.checkpointPath);
    await this.loadGgufAdapter(ggufPath, 'U');
  } else {
    // Reuse existing GGUF (no conversion needed)
    console.log('[QLoRATrainer] Skipping conversion - using existing GGUF');
  }
}

private async shouldConvertToGguf(adapterType: 'U' | 'T'): Promise<boolean> {
  const { Preferences } = await import('@capacitor/preferences');
  const { Filesystem, Directory } = await import('$lib/adapters/filesystem');

  // Check if GGUF exists
  const ggufPath = await Preferences.get({
    key: `lora_adapter_${adapterType}_gguf_path`
  });

  if (!ggufPath.value) {
    return true; // No GGUF exists, need to convert
  }

  // Check if safetensors changed (compare checksums)
  const safetensorsPath = await Preferences.get({
    key: `lora_adapter_${adapterType}_safetensors_path`
  });

  if (!safetensorsPath.value) {
    return false; // No safetensors, can't convert
  }

  const safetensorsChecksum = await this.computeChecksum(safetensorsPath.value);
  const lastChecksum = await Preferences.get({
    key: `lora_adapter_${adapterType}_last_checksum`
  });

  if (safetensorsChecksum !== lastChecksum.value) {
    // Safetensors changed, need to convert
    await Preferences.set({
      key: `lora_adapter_${adapterType}_last_checksum`,
      value: safetensorsChecksum,
    });
    return true;
  }

  return false; // No change, reuse existing GGUF
}
```

---

## Performance Optimization

### Option A: Convert Every Night (Simple)

- **Time**: 1-2 minutes nightly
- **Complexity**: Low
- **Reliability**: High (always up-to-date)

### Option B: Lazy Conversion (Optimized)

- **Time**: 1-2 minutes only when adapter changes
- **Complexity**: Medium (need checksum tracking)
- **Reliability**: Medium (may miss small changes)

**Recommendation**: Start with Option A (convert every night). Optimize to Option B if conversion time becomes an issue.

---

## Summary

**Solution**: Dual-format approach with incremental training

- ✅ **Safetensors**: Keep for incremental training (can continue from previous)
- ✅ **GGUF**: Convert for inference (read-only, but regenerated each night)
- ✅ **Incremental**: Load previous safetensors → Continue training → Save updated safetensors
- ✅ **Conversion**: Happens nightly (1-2 min), regenerates GGUF from updated safetensors
- ✅ **Evolution**: Safetensors adapter evolves incrementally, GGUF is regenerated (not frozen)

**Key Insight**:

- GGUF is "frozen" in the sense that you can't train on it directly
- But we regenerate it each night from the updated safetensors
- The safetensors adapter is the "source of truth" that continues to evolve
- GGUF is just a converted snapshot for inference

**Flow**:

```
Night 1: Train from scratch → safetensors → convert → GGUF
Night 2: Load safetensors → continue training → update safetensors → convert → GGUF
Night 3: Load safetensors → continue training → update safetensors → convert → GGUF
... (adapter continues to evolve incrementally)
```

The adapter is **never frozen** - it continues to evolve in safetensors format. GGUF is just regenerated each night for inference.

## Related

^[{src_rel}]
