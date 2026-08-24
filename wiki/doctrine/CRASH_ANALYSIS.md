---
title: CRASH_ANALYSIS
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/CRASH_ANALYSIS.md
updated: 2026-07-24
---

# SIGABRT Crash Analysis - Root Cause Identified

## Critical Finding: Model Architecture Mismatch

**Issue**: The model file `alice-phi4-mini-q4.gguf` actually contains `phi3` architecture, not `phi4`.

**Evidence**:

- Hexdump shows: `67 65 6e 65 72 61 6c 2e 61 72 63 68 69 74 65 63 74 75 72 65` = "general.architecture"
- Followed by: `70 68 69 33` = "phi3"
- All model parameters reference `phi3.*` attributes

**Root Cause**:

1. Model is misnamed (should be `alice-phi3-mini-q4.gguf`)
2. llama.cpp phi3 implementation has bugs causing SIGABRT during decode
3. Crash occurs at specific memory address in phi3 decode code path

## Immediate Action Plan

### Priority 1: Test with Different Model

1. **Try a known stable model** (Llama-3-8B-Instruct-Q4_K_M.gguf)
2. **Test with actual Phi-4 model** if available
3. **Test with older Phi-3 model** from official sources

### Priority 2: llama.cpp Version Investigation

1. **Check llama.cpp git commit** for phi3-related fixes
2. **Compare with upstream llama.cpp** for phi3 bug fixes
3. **Consider updating llama.cpp** to latest stable

### Priority 3: Model Rebuild

1. **Rebuild the model with correct architecture metadata**
2. **Use official Phi-4 base model** for fine-tuning
3. **Verify model compatibility** before deployment

## Debugging Steps

### Step 1: Model Verification

```bash
# Check actual model architecture
hexdump -C ./models/alice-phi4-mini-q4.gguf | grep -A5 -B5 "phi3"

# Compare with known good model
./main -m ./models/alice-phi4-mini-q4.gguf --test
```

### Step 2: llama.cpp Debug Build

```bash
# Build llama.cpp with debug symbols
cd alicecore_build/llama.cpp
make DEBUG=1

# Run with gdb to get exact crash location
gdb --args ./main -m ./models/alice-phi4-mini-q4.gguf --test
```

### Step 3: Alternative Model Test

```bash
# Test with official Phi-3 model
wget https://huggingface.co/microsoft/Phi-3-mini-4k-instruct-gguf/resolve/main/Phi-3-mini-4k-instruct-q4.gguf

# Test crash reproduction
./main -m Phi-3-mini-4k-instruct-q4.gguf --test
```

## Next Steps

1. **Immediate**: Test with official Phi-3 model to confirm architecture issue
2. **Short-term**: Update llama.cpp to latest version with phi3 fixes
3. **Medium-term**: Rebuild model with correct architecture metadata
4. **Long-term**: Establish model compatibility testing pipeline

## Questions for Team

1. **Model Source**: Where was this `alice-phi4-mini-q4.gguf` model created?
2. **Training Process**: Was it fine-tuned from Phi-3 but mislabeled as Phi-4?
3. **llama.cpp Version**: What specific commit/branch of llama.cpp are we using?
4. **Testing**: Has this model been tested on other platforms successfully?

## Timeline

- **Day 1**: Test with alternative models (2-3 hours)
- **Day 2**: Update llama.cpp and test (4-6 hours)
- **Day 3**: Model rebuild if needed (8-12 hours)
- **Day 4**: Full integration testing (2-3 hours)

## Risk Assessment

**High Risk**:

- Current model is fundamentally incompatible
- All users will experience crashes
- No workaround without model change

**Medium Risk**:

- llama.cpp update may introduce other compatibility issues
- Model rebuild may affect performance/accuracy

**Low Risk**:

- Testing with alternative models is safe
- Can rollback to previous working state if needed

## Related

^[source-materials/mirrors/doctrine/CRASH_ANALYSIS.md]
