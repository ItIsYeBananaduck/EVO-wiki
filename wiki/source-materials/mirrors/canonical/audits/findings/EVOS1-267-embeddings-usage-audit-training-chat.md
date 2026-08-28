---
title: "EVOS1-267 Audit: Existing Embeddings Usage (Training + Chat)"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-05-02
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-267 Audit: Existing Embeddings Usage (Training + Chat)

Date: 2026-05-02  
Issue: EVOS1-267  
Scope: Audit existing embeddings generation, storage, retrieval flow, Training app coupling, and token usage patterns with a primary focus on EVOtraining chat (Alice).

---

## Executive summary

Current production embeddings usage is concentrated in the Flutter Training app’s Alice chat memory pipeline:

- Embeddings are generated on-device via a quantized TFLite MiniLM model (`all-MiniLM-L6-v2-quant.tflite`) through `EmbeddingService`.
- Memory records are stored in local JSONL (`memories.jsonl`) and vector embeddings are stored in a local binary index (`vectors.bin`).
- Retrieval is hybrid: vector similarity + keyword scoring (70% vector / 30% keyword), with graceful fallback to keyword-only when embeddings are unavailable.
- Coupling is currently app-local: memory management + embedding initialization is orchestrated from `AliceBrainService` in the Training app runtime.
- Token usage in chat prompt composition is actively constrained with explicit budgets (e.g., memory brief truncation and mode-specific token caps), which directly affects how much retrieved memory can be injected.

---

## 1) Where embeddings are generated

### Active path

Embeddings are generated in `EmbeddingService`:

- Uses a local TFLite interpreter and BERT tokenizer.
- Model output is pooled/normalized to produce 384-d vectors.
- Single and batch embed APIs exist (`embed`, `embedBatch`).

Evidence:

- `EmbeddingService` class + 384-d comment and embedding API.  
  (`flutter_app/lib/features/alice/domain/embedding_service.dart`)
- `embed()` tokenizes input, executes TFLite inference, and returns normalized embedding vectors.

### Trigger points

Embeddings are generated in two main memory lifecycle points:

1. **On memory write/indexing** (`_indexMemoryAsync`) for new memory entries.
2. **On retrieval query** (`buildMemoryBrief`) to embed the user query for nearest-neighbor search.

Evidence:

- Async embedding during write path (`_indexMemoryAsync -> embeddingService.embed`).
- Query embedding during retrieval (`buildMemoryBrief -> embeddingService.embed(query)`).

---

## 2) Storage locations (vector index + local files)

### Model + tokenizer assets

- Tokenizer vocab is bundled in app assets: `assets/models/embeddings/vocab.txt`.
- Embedding model binary is downloaded (not bundled) and placed under `AliceAssets/models/embeddings/...`.

Evidence:

- Asset declaration includes vocab file in `pubspec.yaml`.
- Asset download manifest includes `all-MiniLM-L6-v2-quant.tflite` and notes R2 download flow.

### Runtime file locations

Conversation memory storage is local-first and per app/user:

- `memories.jsonl` (memory entries)
- `vectors.bin` (vector index)
- `memory_index.json` (auxiliary index metadata)
- `session_summary.json` (summary)

Path root:

- `AliceAssets/memory/<appId>/<userId>/...`

Evidence:

- File path getters and deletion logic in `ConversationMemoryManager` explicitly reference these files.

### SAF/shared store caveat

On Android SAF-backed shared storage, direct file access limitations require copying model bytes into app-doc filesystem for TFLite interpreter use.

Evidence:

- SAF copy bridge logic in `AliceBrainService` before `EmbeddingService.initialize()`.

---

## 3) Retrieval flow (chat memory)

Retrieval flow for chat memory brief:

1. Lazy init vector index on first query.
2. Load memories from local JSONL.
3. Build query keywords (stopword-filtered).
4. If embeddings are ready:
   - Embed query.
   - Vector search against local vector index (`topK` tier-based).
5. Compute hybrid score per memory:
   - 70% vector similarity + 30% normalized keyword score.
6. Select typed memory mix (semantic/procedural/episodic limits).
7. Format and inject as `[MEMORY BRIEF]` into prompt.

Fallback behavior:

- If embeddings/vector path is unavailable or errors, retrieval falls back to keyword scoring only.

Evidence:

- Lazy index init (`_ensureVectorIndexInitialized`).
- Hybrid scoring weights and fallback logic in `buildMemoryBrief`.
- Memory brief formatting and prompt injection path constraints.

---

## 4) Coupling to Training app

Current embeddings + retrieval pipeline is tightly coupled to Training app Alice domain services:

- `AliceBrainService` performs memory manager creation and embedding service initialization.
- It resolves model paths, handles SharedModelStore behavior, and owns fallback/copy logic.
- `ConversationMemoryManager` is created with app-specific identifiers (`appId: 'evo'`, `userId`) and local file layout.

Implication for extraction:

- A shared package can lift core primitives (`EmbeddingService`, `VectorMemoryIndex`, hybrid ranking) but app-level orchestration (storage backend, lifecycle, device-tier policy, prompt assembly) is currently interleaved with Training app concerns.

Evidence:

- Memory manager lazy-init and embedding service wiring in `alice_brain_service.dart`.

---

## 5) Token usage patterns (chat path)

Token budgeting is explicitly enforced in the chat runtime and directly shapes retrieval utility:

- Memory brief truncated to ~400 tokens.
- Tool definitions truncated to ~1000 tokens when needed.
- Base mode guidance enforces strict system/total token targets (e.g., <=200 system, <=600 total in base mode).
- Prompt assembly reserves generation budget from context window.

Why this matters to embeddings migration:

- Retrieval quality is not only index quality; it is also constrained by injection budget.
- Any shared embeddings package migration must keep compatibility with prompt budget policy, or retrieval gains may not surface in responses.

Evidence:

- Prompt budgeting + token compliance logging in `LlamaEngine.swift`.

---

## Risks for extraction into shared embeddings package

1. **Storage abstraction mismatch risk**  
   Current logic branches across direct filesystem and SAF with ad-hoc copy behavior. A shared package needs a stable storage adapter contract.

2. **Lifecycle/ownership coupling risk**  
   Initialization and fallback behavior live in `AliceBrainService`; extracting without preserving initialization order may regress retrieval availability.

3. **Hybrid ranking behavior drift risk**  
   Existing weighting and tier-based topK are product-tuned; changing defaults in shared code can silently alter chat behavior.

4. **Token budget interaction risk**  
   Retrieval output volume is bounded by prompt budgets; shared-layer changes that increase memory text size can reduce effective response quality.

5. **Model/version management risk**  
   Model is remote-downloaded with incomplete checksum enforcement placeholder in asset config; package extraction should enforce model/version/checksum metadata explicitly.

6. **Cross-app namespace risk**  
   Memory pathing currently includes app/user IDs but is local to Training orchestration; shared package should formalize tenant/app namespacing to avoid collisions.

---

## Acceptance criteria mapping (EVOS1-267)

- **All embedding usage locations documented**: Completed (generation service, invocation points, model/tokenizer asset pathing, storage files).
- **Retrieval flow documented**: Completed (lazy init, vector + keyword hybrid scoring, selection + prompt injection, fallback mode).
- **Risks for extraction identified**: Completed (storage, lifecycle, ranking drift, token budget interaction, model governance, namespacing).