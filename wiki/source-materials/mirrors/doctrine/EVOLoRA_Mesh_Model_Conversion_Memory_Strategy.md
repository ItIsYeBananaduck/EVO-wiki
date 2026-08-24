---
title: EVOLoRA_Mesh_Model_Conversion_Memory_Strategy
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_Model_Conversion_Memory_Strategy.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh Model Conversion Memory Strategy

**Problem**: Converting HuggingFace model to MLX format requires significant memory. Need to ensure conversion doesn't exceed available RAM.

**Current System**:

- Total RAM: 16 GB
- Available RAM: ~2.7 GB (low!)
- Disk Space: 2.4 TB available ✅

---

## Memory Requirements

### Model Sizes

| Format                 | Size    | Notes                    |
| ---------------------- | ------- | ------------------------ |
| **HuggingFace (FP16)** | ~7-8 GB | Original format          |
| **MLX (FP16)**         | ~7-8 GB | Converted format         |
| **MLX (Q4)**           | ~2 GB   | Quantized (recommended)  |
| **MLX (Q8)**           | ~4 GB   | Higher quality quantized |

### Conversion Memory Usage

**Peak Memory During Conversion**:

- Download HuggingFace model: ~7-8 GB (temporary)
- Load into MLX: ~7-8 GB (temporary)
- Convert: ~7-8 GB (temporary)
- **Total Peak**: ~8-10 GB (with overhead)

**With Only 2.7 GB Available**: ⚠️ **May fail or be very slow**

---

## Memory-Safe Conversion Strategies

### Strategy 1: Direct Quantization (Recommended)

**Approach**: Convert directly to quantized format, skip FP16 intermediate

**Process**:

1. Convert HuggingFace → MLX (FP16, temporary)
2. **Immediately quantize** to Q4 (reduces to ~2 GB)
3. Delete FP16 intermediate
4. Keep only Q4 model

**Memory Savings**:

- Avoids keeping both FP16 and Q4 in memory
- Reduces peak memory by ~50%

**Script**: `training/export_mlx_base_memory_safe.py` (CREATED)

### Strategy 2: Use Existing MLX Training Output

**Approach**: Check if model is already in MLX format from training

**Check**:

```bash
ls -lh training/alice-phi3-mlx/
```

**If MLX format exists**:

- May already have base model in MLX format
- Can quantize existing MLX model (lower memory)
- Skip HuggingFace download

### Strategy 3: Server-Side Conversion

**Approach**: Convert on server with more memory, upload to R2

**Process**:

1. Convert on server (more RAM available)
2. Upload MLX model to R2
3. Download from R2 to device

**Benefits**:

- No local memory constraints
- Faster (server has more resources)
- Can be done in background

### Strategy 4: Streaming/Chunked Conversion

**Approach**: Convert model in chunks (if MLX supports it)

**Limitation**: MLX's `convert()` may not support chunked conversion

**Alternative**: Use smaller test model first to verify process

---

## Recommended Approach

### Option A: Memory-Safe Script (If Converting Locally)

**Use**: `training/export_mlx_base_memory_safe.py`

**Features**:

- Monitors memory usage
- Converts directly to Q4 (skips FP16 intermediate)
- Forces garbage collection
- Warns if memory is low

**Usage**:

```bash
# Close other applications first to free memory
python training/export_mlx_base_memory_safe.py --quantize q4
```

**Memory Check**:

- Warns if <4 GB available
- Asks for confirmation before proceeding
- Monitors memory during conversion

### Option B: Server-Side Conversion (Recommended)

**Process**:

1. Run conversion on server (more RAM)
2. Upload MLX model to R2
3. Download from R2 (no conversion needed)

**Benefits**:

- No local memory constraints
- Faster conversion
- Can be scheduled/automated

**Script**: Can use same `export_mlx_base_memory_safe.py` on server

### Option C: Use Existing MLX Output

**Check First**:

```bash
ls -lh training/alice-phi3-mlx/
```

**If MLX model exists**:

- May already be converted
- Can quantize existing model (lower memory)
- Skip HuggingFace download

---

## Memory Optimization Tips

### Before Conversion

1. **Close other applications**:
   - Free up as much RAM as possible
   - Target: >4 GB available

2. **Clear caches**:

   ```bash
   # Clear Python cache
   find . -type d -name __pycache__ -exec rm -r {} +

   # Clear system cache (if needed)
   sudo purge  # macOS
   ```

3. **Monitor memory**:
   ```bash
   # Watch memory during conversion
   watch -n 1 'vm_stat | head -20'
   ```

### During Conversion

1. **Use memory-safe script**:
   - `export_mlx_base_memory_safe.py` monitors memory
   - Forces garbage collection
   - Warns if memory is low

2. **Convert directly to Q4**:
   - Skips FP16 intermediate
   - Reduces peak memory by ~50%

3. **Monitor process**:
   - Watch Activity Monitor (macOS)
   - Check for memory pressure
   - Kill if memory usage is too high

---

## Alternative: Check Existing MLX Output

**First, check if model already exists**:

```bash
# Check training output
ls -lh training/alice-phi3-mlx/

# If MLX model exists, we can:
# 1. Use it directly (if already converted)
# 2. Quantize it (lower memory than full conversion)
```

**If MLX model exists**:

- May already be in MLX format
- Can quantize existing model (much lower memory)
- Skip HuggingFace download entirely

---

## Recommended: Server-Side Conversion

**Best Approach**: Convert on server, upload to R2

**Why**:

- ✅ No local memory constraints
- ✅ Faster (server has more resources)
- ✅ Can be automated
- ✅ One-time conversion

**Process**:

1. Run conversion on server (Fly.io, AWS, etc.)
2. Upload MLX model to R2
3. Download from R2 to device (no conversion needed)

**Script**: Same `export_mlx_base_memory_safe.py` can be used on server

---

## Summary

**Current Memory**: 2.7 GB available (low for conversion)

**Options**:

1. ✅ **Memory-safe script**: `export_mlx_base_memory_safe.py` (monitors memory, converts to Q4)
2. ✅ **Server-side conversion**: Convert on server, upload to R2 (recommended)
3. ✅ **Use existing MLX output**: Check if model already exists from training

**Recommendation**:

- **If converting locally**: Use memory-safe script, close other apps, monitor memory
- **If possible**: Convert on server, upload to R2 (no local memory constraints)

**Next Step**: Check if MLX model already exists from training, or proceed with server-side conversion.

## Related
