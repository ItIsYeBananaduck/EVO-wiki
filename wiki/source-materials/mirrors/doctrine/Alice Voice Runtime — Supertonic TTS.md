---
title: Alice Voice Runtime — Supertonic TTS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Alice Voice Runtime — Supertonic TTS.md"]
updated: 2026-07-24
---

# Alice Voice Runtime — Supertonic TTS

> NOTE: This is a canonical architecture note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

---

## Purpose

Define the on-device TTS runtime for Alice voice synthesis using Supertonic 2. Supertonic replaced Kokoro TTS (see [[Alice Voice Runtime — Kokoro TTS]] for deprecated history). This note documents the full implementation as integrated in `SupertonicTtsPlugin.swift`.

---

## Core Principle

Alice's voice lives entirely on-device. No cloud TTS. Supertonic synthesizes speech locally using a 4-model ONNX pipeline without any network dependency. Models are loaded lazily on the first `speak()` call to reduce startup memory pressure.

---

## Definitions

- **Supertonic 2** — the current on-device neural TTS engine; 4-model ONNX pipeline producing 44.1kHz Float32 audio
- **Method channel** — Flutter↔Swift bridge for TTS (`evo/supertonic_tts`)
- **NFE** — Number of Function Evaluations; denoising steps in the vector estimator (default: 5)
- **Emotion tag** — a bracket-tagged modifier appended to LLM output text to shape voice cadence (e.g. `[neutral]`, `[momentumLift]`)
- **tempoScale** — the primary lever for emotion modulation; applied to predicted durations before synthesis
- **Speech reshaping** — pre-TTS text normalization pass that removes corporate phrasing and structures text for natural spoken rhythm

---

## System Structure

### Method Channel Contract

**Channel:** `evo/supertonic_tts`

| Method | Args | Returns |
|---|---|---|
| `isAvailable` | — | `bool` (asset presence check only — does NOT load models) |
| `speak` | `text: String, speed: Double` | playback health map |
| `stop` | — | `bool` |
| `getDiagnosticStatus` | — | `Map` |

**`speak` playback health map:**
```
{
  "ok": Bool,
  "activation": "ok" | "failed" | "not_attempted",
  "sessionDucking": Bool,
  "sessionA2DP": Bool,
  "route": "valid" | "invalid" | "not_checked",
  "routeOutputs": [String],
  "engineStart": "started" | "failed" | "not_started",
  "reason": String
}
```

**`getDiagnosticStatus` map:**
```
{
  "isInitialized": Bool,
  "lastError": String,
  "onnxRuntimeAvailable": Bool,
  "voiceStyle": String,          // always "F2"
  "textEncoderLoaded": Bool,
  "durationPredictorLoaded": Bool,
  "vectorEstimatorLoaded": Bool,
  "vocoderLoaded": Bool
}
```

---

### Model Assets

**Total size:** ~250 MB (vs Kokoro's 156 MB single model)

| File | Size | Role |
|---|---|---|
| `onnx/text_encoder.onnx` | 26.16 MB | Encodes tokenized text into embeddings |
| `onnx/duration_predictor.onnx` | 1.45 MB | Predicts per-token audio durations |
| `onnx/vector_estimator.onnx` | 126.33 MB | Denoising loop (NFE=5 steps by default) |
| `onnx/vocoder.onnx` | 96.71 MB | Converts denoised latent to 44.1kHz PCM |
| `onnx/tts.json` | — | Runtime config (sample rate, chunk sizes, latent dim) |
| `onnx/unicode_indexer.json` | — | Unicode-to-token-ID mapping for `UnicodeProcessor` |
| `voice_styles/F2.json` | — | Voice style embeddings for F2 (Olivia) |

All models use ONNX opset 19. License: OpenRAIL-M (commercial use permitted).

**Asset lookup order (checked in sequence):**
1. App Group: `group.biz.lsctech.adaptivefit/EVO/ModelStore/AliceAssets/supertonic-2/`
2. Documents: `<DocumentsDir>/AliceAssets/supertonic-2/`

Assets must not be bundled in the app binary; pulled at device setup time.

**Runtime config (`tts.json`):**
```json
{
  "ae": { "sample_rate": 44100, "base_chunk_size": 512 },
  "ttl": { "chunk_compress_factor": 6, "latent_dim": 24 }
}
```

---

### Inference Pipeline

**Voice:** F2 (Olivia) — hardcoded
**Language:** `"en"` (English)
**Sample rate:** 44.1kHz Float32 mono
**Default speed:** 1.05 (slightly faster than neutral)

```
LLM output text
    → Speech reshaping pass (reshapeForSpeech)
    → Emotion tag extraction ([neutral|subtleApproval|calmConcern|momentumLift])
    → Text chunking (max 300 chars / chunk)
    → Per-chunk inference:
        1. UnicodeProcessor.call(chunk, lang)               → textIds, textMask
        2. duration_predictor(textIds, style_dp, textMask)  → duration[]
        3. Apply tempoScale to duration[]
        4. text_encoder(textIds, style_ttl, textMask)       → text_emb
        5. sampleNoisyLatent(duration)                      → xt, latentMask
        6. vector_estimator (×NFE denoising steps)          → denoised_latent
        7. vocoder(denoised_latent)                         → wav_tts (Float32 PCM)
    → Cadence silence insertion between chunks
    → AVAudioEngine playback via SharedAudioSessionCoordinator
```

**Denoising loop** runs `defaultNFE = 5` iterations. Each step passes `current_step` and `total_step` scalars alongside `noisy_latent`, `text_emb`, `style_ttl`, `latent_mask`, `text_mask`.

**Noisy latent sampling** uses Box-Muller transform to generate Gaussian noise shaped to `[1, latentDim × chunkCompressFactor, latentLen]`.

---

### Lazy Initialization

Models are **not** loaded at `isAvailable()`. Loading happens on the first `speak()` call. This avoids memory contention with `LlamaEngine` during app startup.

`isAvailable()` only checks that all required asset files are present on disk.

---

### llama.cpp Conflict Handling

When `LlamaEngine.shared.isLoaded()` returns `true`, Supertonic uses **CPU-only ONNX Runtime** to avoid Metal GPU conflicts. When llama.cpp is idle, the **CoreML Execution Provider** is attempted for better performance; falls back to CPU if CoreML EP is unavailable.

---

### Emotion System

LLM output may append one of four emotion tags at the end of the text string. The tag is stripped before synthesis; only the `tempoScale` is applied.

| Tag | tempoScale | Effect |
|---|---|---|
| `[neutral]` | 1.0 | Baseline — steady, grounded delivery |
| `[subtleApproval]` | 1.02 | +2% rate — mild positive energy |
| `[calmConcern]` | 0.98 | −2% rate — slower, grounded tone |
| `[momentumLift]` | 1.03 | +3% rate — forward momentum |

**Guardrails:**
- Responses ≤6 words → always force `[neutral]` (prevents odd coloring on short replies like "Got it.")
- Active workout context + `[momentumLift]` → clamp to `[subtleApproval]` (avoids over-energizing during peak exertion)

Note: `pitchScale` and `energyScale` fields are parsed but not applied — the model architecture does not expose these levers directly.

---

### Speech Reshaping Pass

Applied before tokenization. Purpose: produce text that sounds natural when spoken, not read.

**Step 1 — Remove banned intro phrases:**
`"To "`, `"In order to "`, `"For example, "`, `"If you find yourself "`, `"Here are some tips"`, `"Here are some ideas"`, `"Here's what you can do"`

**Step 2 — Replace dense corporate phrases:**

| Dense | Replaced with |
|---|---|
| "underlying causes" | "the trigger" |
| "implement strategies" | "try this" |
| "address the issue" | "fix it" |
| "it's important to" | *(removed)* |
| "you might want to consider" | "try" |
| "in order to achieve" | "to get" |
| "make sure to" | *(removed)* |
| "don't forget to" | *(removed)* |

**Step 3 — Split sentences >18 words** at natural break points (`, `, ` and `, ` but `, ` so `).

**Step 4 — Short sentence anchor injection:** If no sentence in the output is ≤6 words, prepend a context-aware anchor:
- Contains "slow"/"rest"/"recover" → `"Slow down."`
- Contains "form"/"watch"/"careful" → `"Watch this."`
- Contains "start"/"begin"/"first" → `"Start here."`
- Contains "check"/"notice"/"look" → `"Check this."`
- Default → `"Got it."`

---

### Text Chunking

- Max chunk length: 300 chars (English)
- Split strategy: sentences (`.!?`) → reassembled greedily until limit
- Each chunk is processed independently through the full inference pipeline

---

### Cadence (Inter-chunk Silence)

| Position | Silence |
|---|---|
| After first chunk (i==1) | 0.35s — grounding pause |
| Before final chunk (last, when >2 chunks) | 0.2s — momentum into directive |
| All other positions | 0.3s — default |

---

### Audio Session

Uses `SharedAudioSessionCoordinator.shared.activatePlaybackSession(context:)` — shared coordinator manages category `.playback` / mode `.spokenAudio` to bypass the silent switch and coordinate with other audio consumers (e.g. workout music). Session is deactivated 0.3s after playback completes to allow music to restore normal volume.

Audio engine: `AVAudioEngine` + `AVAudioPlayerNode`. If the engine stops due to interruption, it is restarted before the next `speak()`.

Not available in iOS Simulator (ONNX Runtime is a device-only dependency).

---

### UnicodeProcessor

Handles text tokenization before ONNX inference:
1. NFKD decomposition
2. Remove emojis (Unicode ranges: 0x1F600–0x1F64F, 0x1F300–0x1F5FF, 0x1F680–0x1F6FF, 0x2600–0x26FF)
3. Symbol normalization (em-dashes → `-`, curly quotes → straight, etc.)
4. Strip extra whitespace; append `.` if no terminal punctuation
5. Wrap with language tags: `<en>text</en>`

Maps characters via `unicode_indexer.json` lookup. Unknown code points map to `-1`.

---

## Rules

- LLM output must pass the instruction-leak detection pass before reaching TTS — see [[Alice Voice Runtime — Kokoro TTS]] for the existing leak detection contract (still applies regardless of TTS engine)
- Voice assets must not be bundled in the app binary; pulled at setup time
- Supertonic must not transmit audio data or synthesis requests off-device
- Models are loaded lazily — `initializeOnnxSessions()` must not be called at availability check time
- CPU-only mode is mandatory when `LlamaEngine.shared.isLoaded()` is true

---

## Flow

1. LLM generates response text with optional emotion tag
2. Leak-detection pass in `LlamaEngine.swift` strips any instruction leakage
3. `speak(text:speed:)` called via `evo/supertonic_tts` method channel
4. If not yet initialized → `initializeOnnxSessions()` runs (lazy)
5. `reshapeForSpeech()` → emotion tag extraction → text chunking
6. Per-chunk: UnicodeProcessor → 4-model ONNX inference → Float32 PCM
7. Chunks assembled with cadence silence
8. `SharedAudioSessionCoordinator` activates session → `AVAudioPlayerNode` plays buffer
9. Session deactivated 0.3s after completion

---

## Relationships

See also: [[Alice Voice Spec]], [[EVO On-Device First Principle]], [[Prompt Injection Boundary]], [[Alice Voice Runtime — Kokoro TTS]]

---

## Edge Cases / Special Handling

- Simulator: `isAvailable()` returns false; `speak()` returns `SYNTHESIS_ERROR` — ONNX Runtime is device-only
- Engine interruption (e.g. phone call): `AVAudioEngine` stops; automatically restarted on next `speak()`
- All 4 models must be present for initialization to succeed — partial asset presence is treated as not available
- Empty text: `speak()` returns `true` immediately without synthesis
- Asset update: replace files in the App Group or Documents path; re-initialization happens automatically on next `speak()` after the app restarts

---

## Summary

Supertonic 2 is the current on-device voice synthesis engine for Alice, replacing Kokoro TTS. It runs a 4-model ONNX pipeline (text_encoder → duration_predictor → vector_estimator → vocoder) producing 44.1kHz Float32 audio. Voice is hardcoded to F2 (Olivia). The Flutter↔Swift bridge is `evo/supertonic_tts`. Models load lazily on first use to avoid contention with LlamaEngine. An emotion tag system (`[neutral|subtleApproval|calmConcern|momentumLift]`) modulates speech tempo within ±3%. A pre-synthesis speech reshaping pass removes corporate phrasing and ensures rhythmically natural spoken output. All synthesis is on-device; no network calls.

## Related
