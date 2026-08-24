---
title: QUANTIZE
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/QUANTIZE.md"]
updated: 2026-07-24
---

# Quantize model (LLAMA -> Q4_K_M)

This doc describes how to build the quantizer and convert the existing `models/alice-mistral-tuned.f16.gguf` to a Q4_K_M quantized model.

Recommended: run these steps inside WSL (Ubuntu) or any Linux VM. If you prefer Colab, the same commands will work inside a notebook cell.

1. From the repository root, run the helper script (WSL):

```bash
# Make the script executable first
chmod +x ./scripts/quantize.sh
# If the model is in the repo under models/, just run:
./scripts/quantize.sh
# Or specify input and output explicitly:
./scripts/quantize.sh /mnt/c/Users/Philm/path/to/models/alice-mistral-tuned.f16.gguf
```

2. What the script does:

- clones a fresh shallow copy of `llama.cpp` into `llama.cpp.build` (to avoid accidental collisions)
- configures CMake with `-DLLAMA_CURL=OFF` (safe fallback)
- builds the quantizer
- runs the quantizer with `-t q4_k_m` and writes the output next to the input file unless `-o` is specified

3. Dependencies (install once on WSL/Ubuntu):

```bash
sudo apt update
sudo apt install -y build-essential cmake git python3 python3-pip wget libopenblas-dev libgomp1 libcurl4-openssl-dev
```

4. Colab notes:

- Use the same apt install commands in a cell, then `git clone` and run the script. Upload your `*.f16.gguf` to Colab (Files or Drive) before running.

5. When finished

- The output file will be named `alice-mistral-tuned.Q4_K_M.gguf` (or explicit `-o` path). Transfer it back to Windows using `/mnt/c/...` paths or scp, or upload to a storage location.

If you want, I can:

- prepare a ready-to-run Colab notebook (with cells for apt install, build, and quantize), or
- run a WSL command locally to verify the environment and run the script (you must allow it), or
- create a GitHub release upload step to store the artifact after quantization.

## Related
