---
title: EVOLoRA_Mesh_Model_Conversion_Options
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_Model_Conversion_Options.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh Model Conversion Options

**Current Situation**:

- Available RAM: 2.7 GB (low for full conversion)
- Existing MLX model: `training/alice-phi3-mlx/fused/` (~7 GB, already in MLX format!)
- Need: Quantized MLX model for iOS (~2 GB)

---

## Option 1: Quantize Existing MLX Model (Recommended - Lowest Memory)

**What You Have**: `training/alice-phi3-mlx/fused/` - Already in MLX format!

**Process**: Just quantize the existing model (no conversion needed)

**Memory Usage**: ~2-3 GB peak (much lower than full conversion)

**Script**: `training/quantize_existing_mlx.py` (CREATED)

**Usage**:

```bash
python training/quantize_existing_mlx.py \
    --input training/alice-phi3-mlx/fused \
    --output models/alice-phi3-mlx-base-q4
```

**Benefits**:

- ✅ **Lowest memory usage** (~2-3 GB peak)
- ✅ **Fast** (no download, no conversion)
- ✅ **Uses existing model** (already in MLX format)
- ✅ **Safe** (works with 2.7 GB available)

**Time**: ~5-10 minutes (just quantization)

---

## Option 2: Direct Quantization During Conversion (If Starting Fresh)

**Process**: Convert HuggingFace → MLX with direct quantization (one step)

**Memory Usage**: ~4-5 GB peak (lower than two-step conversion)

**Script**: `training/export_mlx_base.py` (UPDATED with direct quantization)

**Usage**:

```bash
python training/export_mlx_base.py --quantize q4
```

**Benefits**:

- ✅ **Single step** (convert + quantize)
- ✅ **Lower memory** than two-step (skip FP16 intermediate)
- ⚠️ **Still needs ~4-5 GB** (may be tight with 2.7 GB available)

**Time**: ~10-20 minutes (download + convert + quantize)

---

## Option 3: Memory-Safe Two-Step Conversion

**Process**: Convert → Quantize (with memory monitoring)

**Memory Usage**: ~8-10 GB peak (high, may fail with 2.7 GB available)

**Script**: `training/export_mlx_base_memory_safe.py` (CREATED)

**Usage**:

```bash
# Close other apps first to free memory
python training/export_mlx_base_memory_safe.py --quantize q4
```

**Benefits**:

- ✅ **Monitors memory** (warns if low)
- ✅ **Forces garbage collection**
- ⚠️ **High memory usage** (may fail with 2.7 GB available)

**Time**: ~10-20 minutes

---

## Option 4: Server-Side Conversion (Recommended for Low Memory)

**Process**: Convert on server, upload to R2

**Memory Usage**: No local memory constraints

**Script**: Same scripts, run on server

**Benefits**:

- ✅ **No local memory constraints**
- ✅ **Faster** (server has more resources)
- ✅ **Can be automated**

**Time**: ~10-20 minutes (on server)

---

## Recommendation: Use Existing MLX Model

**Best Option**: **Option 1 - Quantize Existing MLX Model**

**Why**:

- ✅ You already have MLX model in `training/alice-phi3-mlx/fused/`
- ✅ Just need to quantize it (lowest memory: ~2-3 GB)
- ✅ Works with your current 2.7 GB available RAM
- ✅ Fast (~5-10 minutes)

**Command**:

```bash
python training/quantize_existing_mlx.py \
    --input training/alice-phi3-mlx/fused \
    --output models/alice-phi3-mlx-base-q4
```

**After Quantization**:

1. Upload to R2: `alice-assets/models/alice-phi3-mlx-base-q4/`
2. Update download manager
3. Build MLX inference engine

---

## Memory Comparison

| Option                  | Peak Memory | Works with 2.7 GB? | Time       |
| ----------------------- | ----------- | ------------------ | ---------- |
| **Quantize Existing**   | ~2-3 GB     | ✅ Yes             | ~5-10 min  |
| **Direct Quantization** | ~4-5 GB     | ⚠️ Maybe           | ~10-20 min |
| **Two-Step Conversion** | ~8-10 GB    | ❌ No              | ~10-20 min |
| **Server-Side**         | N/A         | ✅ Yes             | ~10-20 min |

**Recommendation**: **Quantize existing MLX model** (Option 1)

---

## Next Steps

1. **Quantize existing model**:

   ```bash
   python training/quantize_existing_mlx.py \
       --input training/alice-phi3-mlx/fused \
       --output models/alice-phi3-mlx-base-q4
   ```

2. **Verify output**:

   ```bash
   ls -lh models/alice-phi3-mlx-base-q4/
   # Should be ~2 GB
   ```

3. **Upload to R2**:
   - Upload `models/alice-phi3-mlx-base-q4/` to R2
   - Path: `alice-assets/models/alice-phi3-mlx-base-q4/`

4. **Update download manager**:
   - Add MLX model to download list
   - Platform detection (iOS downloads MLX)

---

## Summary

**You already have the MLX model!** Just quantize it:

- ✅ Lowest memory usage (~2-3 GB)
- ✅ Works with your current RAM
- ✅ Fast (~5-10 minutes)
- ✅ No conversion needed

**Script**: `training/quantize_existing_mlx.py` (ready to use)

## Related

^[{src_rel}]
