---
title: EVOLoRA_Mesh_Nightly_Update_Implementation
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_Nightly_Update_Implementation.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh Nightly Update Implementation

**Solution**: Server-side LoRA training + GGUF conversion + Nightly device download

---

## Architecture Flow

```
┌─────────────────────────────────────────────────────────────┐
│ WEEKLY SERVER-SIDE PIPELINE (Sunday)                        │
├─────────────────────────────────────────────────────────────┤
│ 1. Federated Aggregation (EXISTING)                         │
│    - Aggregate user deltas from past week                  │
│    - Aggregate trainer actions from past week               │
│    - Runs on Sunday (weekly schedule)                       │
│                                                              │
│ 2. Train LoRA Adapters (NEW)                                 │
│    - Train U adapter (per-user, from user deltas)           │
│    - Train T adapter (per-trainer, from trainer actions)    │
│    - Train GU adapter (global user, aggregated)             │
│    - Train GT adapter (global trainer, aggregated)          │
│    - Output: safetensors format                             │
│                                                              │
│ 3. Convert to GGUF (NEW)                                     │
│    - Convert each safetensors → GGUF                        │
│    - Use convert_lora_to_gguf.py                            │
│                                                              │
│ 4. Upload to R2 (NEW)                                        │
│    - Upload GGUF adapters to R2                             │
│    - Update version metadata                                │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ NIGHTLY DEVICE-SIDE CHECK (Every Night at 3 AM)             │
├─────────────────────────────────────────────────────────────┤
│ 1. Background Task Trigger (EXISTING)                        │
│    - backgroundModelUpdater.ts runs at 3 AM                 │
│    - Checks for adapter updates                             │
│                                                              │
│ 2. Check Version (NEW)                                       │
│    - Query adapter version metadata                         │
│    - Compare with locally cached version                    │
│    - If newer version available → download                 │
│    - If no update → skip (most nights)                      │
│                                                              │
│ 3. Download GGUF Adapters (NEW, only when update available) │
│    - Download if newer version detected                     │
│    - Use existing modelUpdater.ts infrastructure            │
│                                                              │
│ 4. Load in llama.cpp (EXISTING)                             │
│    - LlamaEngine.applyAdapterStack()                        │
│    - Ready for inference                                    │
└─────────────────────────────────────────────────────────────┘
```

---

## Implementation Steps

### Step 1: Add LoRA Training to Federated Server

**File**: `federated-server/src/ml/lora_trainer.py` (NEW)

```python
"""
Train LoRA adapters from aggregated federated data
"""
import torch
from peft import LoraConfig, get_peft_model, TaskType
from transformers import AutoModelForCausalLM, AutoTokenizer
from pathlib import Path

async def train_lora_adapter(
    training_data: List[Dict],
    base_model: str,
    output_dir: str,
    adapter_type: str  # "U", "T", "GU", "GT"
) -> str:
    """Train LoRA adapter and return path to safetensors file."""

    # Load base model
    model = AutoModelForCausalLM.from_pretrained(
        base_model,
        torch_dtype=torch.float16,
        device_map="auto"
    )

    # Configure LoRA
    lora_config = LoraConfig(
        r=16,
        lora_alpha=32,
        target_modules=["q_proj", "k_proj", "v_proj", "o_proj"],
        task_type=TaskType.CAUSAL_LM,
    )

    model = get_peft_model(model, lora_config)

    # Train (simplified - full implementation needed)
    # ... training loop ...

    # Save adapter
    model.save_pretrained(output_dir)

    return str(Path(output_dir) / "adapter_model.safetensors")
```

### Step 2: Add GGUF Conversion

**File**: `federated-server/src/ml/lora_converter.py` (NEW)

```python
"""
Convert LoRA safetensors to GGUF format
"""
import subprocess
import sys
from pathlib import Path

async def convert_lora_to_gguf(
    lora_dir: str,
    base_model: str,
    output_path: str
) -> str:
    """Convert PEFT LoRA adapter to GGUF format."""

    # Find convert script (in alicecore_build or clone llama.cpp)
    converter = Path("alicecore_build/llama.cpp/convert_lora_to_gguf.py")

    if not converter.exists():
        raise FileNotFoundError("convert_lora_to_gguf.py not found")

    cmd = [
        sys.executable,
        str(converter),
        lora_dir,
        "--base-model-id", base_model,
        "--outfile", output_path,
        "--outtype", "f16",  # Float16 for balance of size/quality
    ]

    result = subprocess.run(cmd, capture_output=True, text=True, check=True)

    if not Path(output_path).exists():
        raise RuntimeError(f"Conversion failed: {result.stderr}")

    return output_path
```

### Step 3: Add to Weekly Aggregation Pipeline

**File**: `federated-server/src/api/aggregate.py` (MODIFY)

```python
from ..ml.lora_trainer import train_lora_adapter
from ..ml.lora_converter import convert_lora_to_gguf
from ..storage.hf_uploader import upload_to_r2

@router.post("/aggregate/trigger")
async def trigger_aggregation():
    """Trigger WEEKLY aggregation + LoRA training + conversion.

    This runs on Sunday (weekly schedule), not nightly.
    After aggregation completes, adapters are available for device download.
    """
    try:
        job_id = str(uuid4())

        # 1. Existing delta merging (for global patch)
        merge_result = await merge_deltas(job_id)

        # 2. NEW: Train LoRA adapters from aggregated data
        # Get aggregated data from past week
        user_data = await get_aggregated_user_data(week_start=get_week_start())
        trainer_data = await get_aggregated_trainer_data(week_start=get_week_start())

        # Train GU adapter (global user) - weekly update
        gu_lora_dir = f"/tmp/gu_lora_{job_id}"
        gu_safetensors = await train_lora_adapter(
            training_data=user_data,
            base_model="microsoft/Phi-3-mini-4k-instruct",
            output_dir=gu_lora_dir,
            adapter_type="GU"
        )

        # Convert to GGUF
        gu_gguf = await convert_lora_to_gguf(
            lora_dir=gu_lora_dir,
            base_model="microsoft/Phi-3-mini-4k-instruct",
            output_path=f"{gu_lora_dir}.gguf"
        )

        # Upload to R2
        await upload_to_r2(
            local_path=gu_gguf,
            r2_key="alice-assets/adapters/global/user/global_user_lora.gguf",
            version=job_id
        )

        # Update version metadata (device will detect this on next nightly check)
        await update_adapter_version(
            adapter_type="GU",
            version=job_id,
            checksum=compute_checksum(gu_gguf),
            r2_path="alice-assets/adapters/global/user/global_user_lora.gguf"
        )

        # Repeat for GT adapter (global trainer)
        # ... similar process ...

        return {
            "success": True,
            "job_id": job_id,
            "merge_result": merge_result,
            "adapters_trained": ["GU", "GT"],
            "message": "Weekly aggregation complete. Adapters available for device download.",
        }

    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Aggregation failed: {str(e)}")
```

### Step 4: Add Adapter Version Tracking

**File**: `federated-server/src/storage/supabase_client.py` (MODIFY)

```python
async def update_adapter_version(
    adapter_type: str,  # "U", "T", "GU", "GT"
    version: str,
    checksum: str,
    r2_path: str
):
    """Update adapter version metadata in Supabase."""
    supabase = supabase_client()

    result = supabase.table("lora_adapter_versions").upsert({
        "adapter_type": adapter_type,
        "version": version,
        "checksum": checksum,
        "r2_path": r2_path,
        "published_at": datetime.utcnow().isoformat() + "Z",
    }).execute()

    return result
```

### Step 5: Device-Side Download Integration

**File**: `app/src/lib/services/ml/modelUpdater.ts` (MODIFY)

```typescript
// Add adapter update check
async checkForAdapterUpdates(): Promise<void> {
  // Check version metadata
  const currentVersions = await this.getCurrentAdapterVersions();
  const latestVersions = await this.fetchLatestAdapterVersions();

  for (const [type, latest] of Object.entries(latestVersions)) {
    const current = currentVersions[type];
    if (latest.version !== current?.version) {
      // Download new adapter
      await this.downloadAdapter(type, latest);
    }
  }
}

async downloadAdapter(type: string, metadata: AdapterMetadata): Promise<void> {
  // Use existing download infrastructure
  const adapterPath = `alice-assets/adapters/${this.getAdapterPath(type)}`;

  await this.downloadFromR2({
    r2Key: metadata.r2_path,
    localPath: adapterPath,
    expectedSize: metadata.size_bytes,
    checksum: metadata.checksum,
  });
}
```

### Step 6: Integrate with Nightly Background Task

**File**: `app/src/lib/services/ml/backgroundModelUpdater.ts` (MODIFY)

```typescript
private async performNightlyCheck(trigger: string): Promise<void> {
  // ... existing model update code ...

  // NEW: Check for adapter updates (runs nightly, but updates only available weekly)
  try {
    const updateResult = await modelUpdater.checkForAdapterUpdates();
    if (updateResult.hasUpdates) {
      console.log(`Nightly adapter check: Found ${updateResult.updatedCount} adapter updates`);
    } else {
      console.log('Nightly adapter check: No updates available (weekly aggregation not yet complete)');
    }
  } catch (error) {
    console.error('Error during adapter update check:', error);
    // Don't fail entire nightly check if adapter check fails
  }
}
```

**Note**: Device checks **nightly**, but adapters are only updated **weekly** after server-side aggregation. Most nights, the check will find no updates (which is fine - it's a lightweight version check).

---

## Update Frequency

| Adapter                 | Update Frequency | Trigger                                     |
| ----------------------- | ---------------- | ------------------------------------------- |
| **U** (User)            | Weekly           | After user's weekly federated aggregation   |
| **T** (Trainer)         | Weekly           | After trainer's weekly action aggregation   |
| **GU** (Global User)    | **Weekly**       | After weekly federated aggregation (Sunday) |
| **GT** (Global Trainer) | **Weekly**       | After weekly trainer aggregation (Sunday)   |

**Note**: Server-side aggregation is **weekly** (not nightly). Device checks for updates **nightly**, but will only find new versions after weekly aggregation completes.

---

## Database Schema

**New Table**: `lora_adapter_versions`

```sql
CREATE TABLE lora_adapter_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  adapter_type TEXT NOT NULL CHECK (adapter_type IN ('U', 'T', 'GU', 'GT')),
  user_id UUID REFERENCES users(id),  -- NULL for global adapters
  trainer_id UUID REFERENCES users(id),  -- NULL for user adapters
  version TEXT NOT NULL,
  checksum TEXT NOT NULL,
  r2_path TEXT NOT NULL,
  size_bytes BIGINT,
  published_at TIMESTAMPTZ NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(adapter_type, user_id, trainer_id, version)
);

-- Index for fast version lookups
CREATE INDEX idx_adapter_versions_lookup
  ON lora_adapter_versions(adapter_type, user_id, trainer_id, published_at DESC);
```

---

## Key Points

1. **Server-side aggregation is WEEKLY** (Sunday) - Not nightly
2. **Conversion happens server-side** - Fast, efficient, no device overhead
3. **Device checks NIGHTLY** - But only downloads when weekly update is available
4. **Device downloads GGUF** - Pre-converted, ready to load
5. **Integrates with existing infrastructure** - Uses `backgroundModelUpdater.ts`, `modelUpdater.ts`
6. **Version tracking** - Prevents unnecessary downloads (most nights, check finds no update)

**Timeline**:

- **Sunday**: Server aggregates data → Trains adapters → Converts to GGUF → Uploads to R2
- **Monday-Saturday**: Device checks nightly, finds no updates (expected)
- **Monday (after Sunday aggregation)**: Device checks, finds new version → Downloads

---

## Next Steps

1. ✅ Create `lora_trainer.py` for training adapters
2. ✅ Create `lora_converter.py` for GGUF conversion
3. ✅ Integrate into aggregation pipeline
4. ✅ Add version tracking to Supabase
5. ✅ Update device download logic
6. ✅ Test end-to-end pipeline

---

## Testing

1. **Server-side**: Test training + conversion pipeline
2. **Device-side**: Test adapter download and loading
3. **Integration**: Test nightly update flow
4. **Performance**: Measure conversion time, download size, load time

## Related

^[{src_rel}]
