---
title: ENF_DOWNLOAD_ISSUE
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/ENF_DOWNLOAD_ISSUE.md
updated: 2026-07-24
---

# ENF LoRA Download Issue

**Status**: ⚠️ Permission/Rate Limit Issue

The ENF LoRA folder at [this link](https://drive.google.com/drive/folders/12IJjSqQ4vNd_KotOHO2kJ_ziX-TwexZo?usp=share_link) is having download issues.

## Problem

- Folder structure is detected correctly
- Files are listed (checkpoint-500, 1000, 1500, 2000 all have `adapter_model.safetensors`)
- But downloads fail with: "Cannot retrieve the public link"

## Solutions

### Option 1: Manual Download via Browser (Easiest)

1. Open the folder: https://drive.google.com/drive/folders/12IJjSqQ4vNd_KotOHO2kJ_ziX-TwexZo
2. Navigate to `enf_lora/checkpoint-2000/`
3. Download `adapter_model.safetensors` (and `adapter_config.json`)
4. Save to: `training/enf_lora/output/downloads/enf_lora_manual/enf_lora/checkpoint-2000/`

Then convert:

```bash
cd training/enf_lora
python3 scripts/convert_to_gguf.py \
  output/downloads/enf_lora_manual/enf_lora/checkpoint-2000/adapter_model.safetensors \
  --base-model ../../training/alice-phi3-mlx/alice-phi3-q4.gguf \
  --output output/gguf/enforcer_lora.gguf \
  --out-type f16
```

### Option 2: Fix Folder Permissions

1. Open the Google Drive folder
2. Click "Share" → "Get link"
3. Set to "Anyone with the link can view"
4. Make sure "Viewer" permission is set (not "Restricted")
5. Try download again

### Option 3: Use Google Drive Desktop App

1. Install Google Drive Desktop app
2. Sync the folder to your computer
3. Files will be available locally
4. Copy to `output/downloads/enf_lora_synced/`

### Option 4: Download Individual Checkpoints

If you only need checkpoint-2000 (most trained):

1. Open: https://drive.google.com/drive/folders/12IJjSqQ4vNd_KotOHO2kJ_ziX-TwexZo
2. Go to: `enf_lora` → `checkpoint-2000`
3. Download these files:
   - `adapter_model.safetensors` ⭐ (most important)
   - `adapter_config.json`
   - `tokenizer_config.json` (optional)
   - `tokenizer.model` (optional)

## What We Know

From the download attempt, we can see the folder contains:

```
enf_lora/
├── checkpoint-500/
│   ├── adapter_model.safetensors ✅
│   ├── adapter_config.json
│   └── ... (other files)
├── checkpoint-1000/
│   ├── adapter_model.safetensors ✅
│   └── ...
├── checkpoint-1500/
│   ├── adapter_model.safetensors ✅
│   └── ...
├── checkpoint-2000/
│   ├── adapter_model.safetensors ✅ (recommended)
│   └── ...
└── (root level files)
    ├── adapter_model.safetensors ✅
    └── ...
```

## Recommended Checkpoint

**Use checkpoint-2000** - it's the most trained (2000 steps).

## After Manual Download

Once you have the files, conversion is ready:

```bash
cd training/enf_lora

# Make sure files are in the right place
mkdir -p output/downloads/enf_lora_manual/enf_lora/checkpoint-2000

# Copy your downloaded files there, then:
python3 scripts/convert_to_gguf.py \
  output/downloads/enf_lora_manual/enf_lora/checkpoint-2000/adapter_model.safetensors \
  --base-model ../../training/alice-phi3-mlx/alice-phi3-q4.gguf \
  --output output/gguf/enforcer_lora.gguf \
  --out-type f16
```

## File IDs (for reference)

If you want to try direct downloads later:

- checkpoint-2000 adapter: `1a1G1ltZPwbgZJkjgIeNLb-BRtTmow_Te`
- checkpoint-1500 adapter: `1AXHgTXOGbeTkuBCEvy79y-TC2aE9E3iQ`
- checkpoint-1000 adapter: `1EK1zfzC89YS8C-lNdq89zs6-X2bcW-3n`
- checkpoint-500 adapter: `1yumYRDQ-udK_I4rQy7u17oRGKHXwS9NW`
- Root adapter: `1Exd6EdVK4gnpSykim9BkLJCyZS3Th6h_`

## Next Steps

1. **Download manually** (Option 1 above) - fastest solution
2. **Fix permissions** (Option 2) - if you control the Drive folder
3. **Convert** using the script once files are local
4. **Upload to R2** along with VOICE LoRA

The conversion script is ready and tested - it just needs the files!

## Related

^[source-materials/mirrors/doctrine/ENF_DOWNLOAD_ISSUE.md]
