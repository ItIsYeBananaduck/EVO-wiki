---
title: EVOLoRA_Mesh_Implementation_Plan
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_Implementation_Plan.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh Implementation Plan

**Status**: Non-Compliant → Target: 100% Compliance
**Date**: 2025-01-08
**Priority**: High (Core Architecture)

---

## Executive Summary

EVOLoRA Mesh is a decision routing system that selects which LoRA adapter (U/T/GU/GT) influences Alice's decisions. Currently, the **Mesh Engine exists** but is **not integrated** with the inference pipeline. This plan addresses all compliance gaps identified in the audit.

---

## Current State Analysis

### ✅ What Exists

- **Mesh Engine** (`flutter_app/lib/features/evolora_mesh/mesh_engine.dart`)
  - Policy weights defined
  - Relevance calculator
  - Availability policy
  - Decision logging
- **Mesh Contexts** (8 contexts defined)
- **Account Descriptor** (user/trainer distinction)
- **Basic LoRA infrastructure** (adapter stack metadata in `alice_brain_service.dart`)

### ❌ What's Missing

#### 1. **Multi-Adapter System** (Critical)

- Only `userLoRA` and `globalPatch` exist
- Missing: `T` (Trainer), `GU` (Global User), `GT` (Global Trainer) adapters
- No adapter download/storage system for multiple adapters
- No adapter loading in llama.cpp

#### 2. **LoRA Router Integration** (Critical)

- MeshEngine exists but is **never called** during inference
- No integration with `LlamaEngine.swift`
- `adapterStack` is metadata only, not actual LoRA files
- No llama.cpp LoRA loading API calls

#### 3. **Plan Versioning** (High)

- No `PlanArtifact` table/model
- No immutable versioning system
- No `plan_id`/`plan_version` in workout logs
- Plans are mutable JSON in SharedPreferences

#### 4. **Weekly Reports** (High)

- No report generation pipeline
- No encrypted report envelope
- No trainer ingestion system
- No deduplication

#### 5. **Trainer Pattern Mining** (Medium)

- No trainer-side mining pipeline
- No aggregate detection
- No pattern-based proposal ordering

#### 6. **RAG Scope Changes** (Medium)

- Current: Daily artifacts with raw logs
- Required: Weekly memory cards with summaries only
- Missing: 12-24 month retention policy

---

## Implementation Plan

### Phase 1: Multi-Adapter Infrastructure (Weeks 1-2)

#### 1.1 Adapter Storage & Download System

**Files to Create/Modify:**

- `flutter_app/lib/features/alice/domain/lora_adapter_manager.dart` (NEW)
- `flutter_app/lib/features/alice/domain/alice_asset_download_manager.dart` (MODIFY)

**Tasks:**

- [ ] Define adapter file naming convention:
  - `user_lora.gguf` (U adapter)
  - `trainer_lora.gguf` (T adapter - per trainer)
  - `global_user_lora.gguf` (GU adapter)
  - `global_trainer_lora.gguf` (GT adapter)
- [ ] Extend `AliceAssetDownloadManager` to download adapters from R2
- [ ] Add adapter version tracking (similar to `trainerAdapterMetadata`)
- [ ] Implement adapter cache management (disk space limits)
- [ ] Add adapter integrity checks (checksums)

**Storage Structure:**

```
AliceAssets/
├── models/
│   └── alice-phi3-q4.gguf (base model)
├── adapters/
│   ├── user/
│   │   └── user_lora.gguf
│   ├── trainer/
│   │   └── {trainerId}_lora.gguf
│   ├── global/
│   │   ├── global_user_lora.gguf
│   │   └── global_trainer_lora.gguf
```

#### 1.2 llama.cpp LoRA Loading API

**Files to Modify:**

- `flutter_app/ios/Runner/LlamaEngine.swift` (ADD LoRA loading)
- `flutter_app/ios/Runner/AppDelegate.swift` (ADD adapter management)

**Tasks:**

- [x] ✅ **Verified**: llama.cpp has LoRA API (`llama_adapter_lora_init`, `llama_set_adapter_lora`)
- [ ] Add `loadLoRAAdapter(path: String) -> OpaquePointer?` method (returns `llama_adapter_lora*`)
- [ ] Add `applyLoRAAdapter(adapter: OpaquePointer, scale: Float) -> Bool` method
- [ ] Add `removeLoRAAdapter(adapter: OpaquePointer) -> Bool` method
- [ ] Add `clearAllLoRAAdapters()` method
- [ ] Add adapter stack management (load multiple adapters, apply with different scales)
- [ ] Add adapter switching logic (unload old, load new based on Mesh decision)

**llama.cpp LoRA API (Verified):**

```c
// Load adapter from file (call after model load, before context creation)
llama_adapter_lora* llama_adapter_lora_init(llama_model* model, const char* path_lora);

// Apply adapter to context with scale (can call multiple times for multiple adapters)
int32_t llama_set_adapter_lora(llama_context* ctx, llama_adapter_lora* adapter, float scale);

// Remove specific adapter
int32_t llama_rm_adapter_lora(llama_context* ctx, llama_adapter_lora* adapter);

// Remove all adapters
void llama_clear_adapter_lora(llama_context* ctx);
```

**API Design:**

```swift
class LlamaEngine {
    private var loadedAdapters: [LoRAAdapter] = []

    func loadLoRAAdapter(
        path: String,
        scale: Float = 1.0,
        adapterId: String
    ) -> Bool

    func applyAdapterStack(_ stack: [LoRAAdapter]) -> Bool

    struct LoRAAdapter {
        let path: String
        let scale: Float
        let kind: LoRAKind  // U, T, GU, GT
    }
}
```

---

### Phase 2: Mesh Router Integration (Weeks 2-3)

#### 2.1 Connect MeshEngine to Inference Pipeline

**Files to Modify:**

- `flutter_app/lib/features/alice/domain/alice_brain_service.dart` (MODIFY)
- `flutter_app/lib/features/alice/domain/alice_mesocycle_service.dart` (MODIFY)
- `flutter_app/lib/features/alice/presentation/alice_chat_screen.dart` (MODIFY)

**Tasks:**

- [ ] Create `MeshRouter` service that wraps `MeshEngine`
- [ ] Before each `generate()` call:
  1. Determine `MeshContext` from domain/action
  2. Call `MeshEngine.select()` or `MeshEngine.blend()`
  3. Get winning LoRA adapter(s)
  4. Build adapter stack with weights
  5. Pass to native layer
- [ ] Integrate with existing decision points:
  - `weeklyOverloadDecision` → `AliceMesocycleService`
  - `safetySubstitution` → `SafetySubstitutionService`
  - `executionMicroadjust` → Live workout adjustments
  - `planCreateMajor` → Plan creation
  - `deloadForcedStop` → Deload detection

**Context Mapping:**

```dart
MeshContext _determineContext(AliceCoachingDomain domain, String action) {
  switch (domain) {
    case AliceCoachingDomain.workout:
      if (isActiveWorkout) return MeshContext.executionMicroadjust;
      return MeshContext.weeklyOverloadDecision;
    case AliceCoachingDomain.planning:
      return MeshContext.planCreateMajor;
    // ...
  }
}
```

#### 2.2 Native Adapter Stack Application

**Files to Modify:**

- `flutter_app/ios/Runner/LlamaEngine.swift` (MODIFY `generate()`)
- `flutter_app/ios/Runner/AppDelegate.swift` (MODIFY `handleAliceBrainCall`)

**Tasks:**

- [ ] Modify `generate()` to accept `adapterStack` with actual paths
- [ ] Before inference:
  1. Unload all current adapters
  2. Load adapters from stack (in priority order)
  3. Apply scales/weights
- [ ] After inference: Keep adapters loaded (for session continuity)
- [ ] Add adapter loading timing metrics

**Flow:**

```
Flutter: MeshEngine.select() → [U: 0.7, GU: 0.3]
Flutter: Build adapter stack → [
  {path: "user_lora.gguf", scale: 0.7, kind: U},
  {path: "global_user_lora.gguf", scale: 0.3, kind: GU}
]
Native: loadLoRAAdapter() for each
Native: Run inference with adapters applied
```

---

### Phase 3: Plan Versioning System (Weeks 3-4)

#### 3.1 PlanArtifact Model & Storage

**Files to Create:**

- `flutter_app/lib/features/alice/domain/plan_artifact.dart` (NEW)
- `flutter_app/lib/features/alice/domain/plan_version_store.dart` (NEW)
- `supabase/migrations/XXX_create_plan_artifacts.sql` (NEW)

**Tasks:**

- [ ] Create `PlanArtifact` model:
  ```dart
  class PlanArtifact {
    final String planId;        // Stable ID
    final int version;          // Monotonic version
    final String hash;          // Canonical JSON hash
    final Map<String, dynamic> workoutPlan;  // Immutable JSON
    final DateTime createdAt;
    final String? trainerId;
    final PlanSource source;    // user, trainer, system
  }
  ```
- [ ] Create Supabase table:
  ```sql
  CREATE TABLE plan_artifacts (
    plan_id TEXT NOT NULL,
    version INTEGER NOT NULL,
    hash TEXT NOT NULL,
    workout_plan JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL,
    trainer_id UUID REFERENCES users(id),
    source TEXT CHECK (source IN ('user', 'trainer', 'system')),
    PRIMARY KEY (plan_id, version)
  );
  ```
- [ ] Migrate existing `TrainingPlanRecord` to use `PlanArtifact`
- [ ] Add version increment logic on plan changes

#### 3.2 Workout Log Linkage

**Files to Modify:**

- `flutter_app/lib/features/intensity/domain/intensity_models.dart` (MODIFY)
- `supabase/migrations/XXX_add_plan_linkage_to_workouts.sql` (NEW)

**Tasks:**

- [ ] Add to `OnDeviceWorkoutLog`:
  ```dart
  final String? planId;
  final int? planVersion;
  final String? dayId;  // Which day in the plan
  ```
- [ ] Add to Supabase `workout_sessions`:
  ```sql
  ALTER TABLE workout_sessions
    ADD COLUMN plan_id TEXT,
    ADD COLUMN plan_version INTEGER,
    ADD COLUMN day_id TEXT;
  ```
- [ ] Update workout logging to capture plan context
- [ ] Add divergence computation (intended vs executed)

---

### Phase 4: Weekly Report Pipeline (Weeks 4-5)

#### 4.1 Report Generation

**Files to Create:**

- `flutter_app/lib/features/alice/domain/weekly_report_generator.dart` (NEW)
- `flutter_app/lib/features/alice/domain/weekly_report_models.dart` (NEW)

**Tasks:**

- [ ] Create `WeeklyReport` model:
  ```dart
  class WeeklyReport {
    final String reportId;
    final String userId;
    final LocalDateRange weekRange;
    final Map<String, dynamic> summary;  // Aggregated stats
    final List<ExerciseSummary> exercises;
    final String payloadHash;  // Canonical hash
    final DateTime generatedAt;
  }
  ```
- [ ] Implement report generation:
  - Aggregate workout logs for the week
  - Compute volume, intensity, adherence metrics
  - Identify patterns (PRs, missed sessions, etc.)
  - Generate summary text
- [ ] Add weekly trigger (Sunday midnight)
- [ ] Store report locally before sending

#### 4.2 Encrypted Report Envelope

**Files to Create:**

- `flutter_app/lib/features/chat/domain/report_envelope.dart` (NEW)

**Tasks:**

- [ ] Create report envelope with:
  - `report_id` (UUID)
  - `payload_hash` (SHA-256 of canonical JSON)
  - `signature` (HMAC or ECDSA signature)
  - `encrypted_payload` (AES-GCM encrypted report)
- [ ] Integrate with existing chat encryption (`chat_crypto.dart`)
- [ ] Send via `ChatSyncService` to trainer

#### 4.3 Trainer Ingestion

**Files to Create:**

- `flutter_app/lib/features/chat/domain/trainer_report_ingestion_service.dart` (MODIFY - exists but incomplete)
- `supabase/migrations/XXX_create_weekly_reports_table.sql` (NEW)

**Tasks:**

- [ ] Create `weekly_reports` table:
  ```sql
  CREATE TABLE weekly_reports (
    report_id TEXT PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    trainer_id UUID REFERENCES users(id),
    week_start DATE NOT NULL,
    payload_hash TEXT NOT NULL UNIQUE,  -- Deduplication
    encrypted_payload BYTEA NOT NULL,
    verified_at TIMESTAMPTZ,
    ingested_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL
  );
  ```
- [ ] Implement verification:
  - Verify signature
  - Check payload hash (deduplication)
  - Validate report structure
- [ ] Store verified reports
- [ ] Index for trainer queries

---

### Phase 5: Trainer Pattern Mining (Weeks 5-6)

#### 5.1 Pattern Detection Pipeline

**Files to Create:**

- `flutter_app/lib/features/alice/domain/trainer_pattern_miner.dart` (NEW)

**Tasks:**

- [ ] Implement per-user pattern detection:
  - Analyze weekly reports over 4+ weeks
  - Detect trends (volume progression, plateaus, etc.)
  - Identify correlations (sleep → performance, etc.)
- [ ] Implement cross-client aggregation:
  - Aggregate patterns across all trainer's clients
  - Compute common patterns
  - Build pattern library
- [ ] Store patterns in local database
- [ ] Use patterns to bias proposal ordering (not override constraints)

#### 5.2 Integration with Mesh

**Files to Modify:**

- `flutter_app/lib/features/evolora_mesh/policy_weights.dart` (MODIFY)

**Tasks:**

- [ ] Add pattern-based weight adjustments
- [ ] Patterns influence `GT` (Global Trainer) relevance
- [ ] Never override safety constraints
- [ ] Log pattern usage in decision records

---

### Phase 6: RAG Scope Refactoring (Weeks 6-7)

#### 6.1 Weekly Memory Cards

**Files to Modify:**

- `VectorRAG/VectorRAG.ts` (MODIFY)
- `VectorRAG/localStore.ts` (MODIFY)

**Tasks:**

- [ ] Change from daily artifacts to weekly:
  - Aggregate 7 days of logs
  - Generate summary (not raw logs)
  - Create tags/rollups
  - Store as "Weekly Memory Card"
- [ ] Schema change:
  ```typescript
  interface WeeklyMemoryCard {
    clientId: string;
    weekStart: string; // ISO date
    summary: string;
    tags: string[];
    metrics: {
      volume: number;
      intensity: number;
      adherence: number;
    };
    vector: number[]; // Embedding
  }
  ```
- [ ] Update retention: 12-24 months (configurable)
- [ ] Add quarterly compression (archive old cards)

#### 6.2 Trainer RAG Queries

**Files to Modify:**

- `VectorRAG/VectorRAG.ts` (MODIFY query methods)

**Tasks:**

- [ ] Add client filter to queries
- [ ] Add time range filter
- [ ] Add tag-based filtering
- [ ] Implement semantic search (vector similarity)
- [ ] Return ranked results

---

## Integration Points

### Critical Integration: Mesh → llama.cpp

**Current Flow:**

```
Flutter: generate() → Native: generate() → llama.cpp: inference
```

**Required Flow:**

```
Flutter: generate()
  → MeshRouter.determineContext()
  → MeshEngine.select() / blend()
  → Build adapter stack [U: 0.7, GU: 0.3]
  → Native: applyAdapterStack()
    → llama.cpp: loadLoRAAdapter() for each
  → Native: generate() with adapters
  → llama.cpp: inference with adapters applied
```

### Key Files to Modify

1. **`alice_brain_service.dart`**
   - Add `MeshRouter` instance
   - Call router before `generate()`
   - Pass adapter stack to native

2. **`LlamaEngine.swift`**
   - Add LoRA loading methods
   - Modify `generate()` to apply adapters
   - Add adapter lifecycle management

3. **`alice_mesocycle_service.dart`**
   - Use `MeshContext.weeklyOverloadDecision`
   - Pass context to MeshRouter

4. **`safety_substitution_service.dart`**
   - Use `MeshContext.safetySubstitution`
   - Already has MeshEngine but needs integration

---

## Testing Strategy

### Unit Tests

- [ ] MeshEngine selection logic
- [ ] Policy weights calculation
- [ ] Availability policy
- [ ] Relevance calculator
- [ ] Adapter stack building

### Integration Tests

- [ ] Mesh → Native adapter loading
- [ ] Multiple adapters loaded simultaneously
- [ ] Adapter switching mid-session
- [ ] Plan versioning workflow
- [ ] Weekly report generation → encryption → ingestion

### End-to-End Tests

- [ ] User workout → Mesh selects U adapter → Inference
- [ ] Trainer creates plan → Mesh selects T adapter → Inference
- [ ] Weekly report → Encrypted → Trainer receives → Ingestion
- [ ] Plan change → Version increment → Workout logs linked

---

## Dependencies & Prerequisites

### llama.cpp Requirements

- [ ] Verify LoRA API exists (`llama_model_apply_lora_from_file`)
- [ ] If not, research alternative (merge adapters pre-inference?)
- [ ] Test LoRA loading performance

### Storage Requirements

- [ ] Estimate disk space for 4 adapters per user
- [ ] Implement adapter cache eviction
- [ ] Add storage quota management

### Backend Requirements

- [ ] R2 bucket structure for adapters
- [ ] Adapter versioning/update system
- [ ] Weekly report storage (Supabase)
- [ ] Plan artifact storage (Supabase)

---

## Success Metrics

### Phase 1 (Adapters)

- ✅ 4 adapter types can be downloaded
- ✅ Adapters load successfully in llama.cpp
- ✅ Adapter switching works without crashes

### Phase 2 (Router)

- ✅ MeshEngine called before every inference
- ✅ Correct adapter selected for each context
- ✅ Decision logs capture all selections

### Phase 3 (Versioning)

- ✅ Plans have immutable versions
- ✅ Workout logs link to plan versions
- ✅ Divergence computation works

### Phase 4 (Reports)

- ✅ Weekly reports generated automatically
- ✅ Reports encrypted and sent to trainer
- ✅ Trainer can ingest and verify reports

### Phase 5 (Mining)

- ✅ Patterns detected per user
- ✅ Cross-client patterns aggregated
- ✅ Patterns influence GT adapter relevance

### Phase 6 (RAG)

- ✅ Weekly memory cards replace daily artifacts
- ✅ Retention policy enforced
- ✅ Trainer queries work with filters

---

## Risk Mitigation

### Risk 1: llama.cpp LoRA API Missing

**Status**: ✅ **RESOLVED** - API exists and verified in headers
**Verified APIs**:

- `llama_adapter_lora_init()` - Load adapter
- `llama_set_adapter_lora()` - Apply with scale
- `llama_rm_adapter_lora()` - Remove adapter
- `llama_clear_adapter_lora()` - Clear all

**Note**: Adapters must be loaded **after model load, before context creation** (per API docs)

### Risk 2: Performance Impact

**Mitigation**:

- Load adapters once per session (not per message)
- Cache loaded adapters
- Profile adapter loading time
- Consider lazy loading (load on first use)

### Risk 3: Storage Bloat

**Mitigation**:

- Implement adapter cache limits (e.g., 2GB max)
- Evict least-recently-used adapters
- Compress adapters if possible
- Stream adapters on-demand

### Risk 4: Backward Compatibility

**Mitigation**:

- Support legacy single-adapter mode
- Feature flag for Mesh routing
- Gradual rollout (beta users first)

---

## Timeline Estimate

| Phase                       | Duration | Dependencies                |
| --------------------------- | -------- | --------------------------- |
| Phase 1: Multi-Adapter      | 2 weeks  | llama.cpp LoRA API research |
| Phase 2: Router Integration | 1 week   | Phase 1 complete            |
| Phase 3: Plan Versioning    | 1 week   | None                        |
| Phase 4: Weekly Reports     | 1 week   | Chat encryption exists      |
| Phase 5: Pattern Mining     | 1 week   | Phase 4 complete            |
| Phase 6: RAG Refactor       | 1 week   | None                        |

**Total**: ~7 weeks (with parallel work on Phases 3 & 6)

---

## Next Steps

1. **Immediate**: Research llama.cpp LoRA API capabilities
2. **Week 1**: Start Phase 1 (adapter infrastructure)
3. **Week 2**: Begin Phase 2 (router integration) in parallel
4. **Week 3**: Start Phase 3 & 6 (can run in parallel)
5. **Week 4**: Phase 4 (weekly reports)
6. **Week 5**: Phase 5 (pattern mining)

---

## Questions to Resolve

1. **Where do trainer adapters come from?**
   - Per-trainer LoRA training pipeline?
   - Or single T adapter for all trainers?
   - **Recommendation**: Start with single T adapter, add per-trainer later

2. **How are GU/GT adapters updated?**
   - Weekly federated aggregation?
   - Manual updates?
   - Automatic from server?
   - **Recommendation**: Weekly federated aggregation (already have infrastructure)

3. **Adapter file format?**
   - ✅ **Verified**: llama.cpp uses GGUF format for LoRA adapters
   - File extension: `.gguf` (same as base model)
   - Loaded via `llama_adapter_lora_init(model, path)`

4. **Trainer-side Alice location?**
   - Same Flutter app (trainer mode)?
   - Web console?
   - Separate trainer app?
   - **Current**: Flutter app supports trainer role (`AppUser.role == 'trainer'`)

5. **Adapter loading timing?**
   - **Critical**: API docs say adapters must be loaded **after model, before context**
   - **Impact**: May need to reload context when switching adapters
   - **Alternative**: Load all adapters upfront, apply/remove as needed (better performance)

---

## References

- Compliance Report: `docs/audits/EVOLoRA_Mesh_Compliance_Report.md`
- Mesh Engine: `flutter_app/lib/features/evolora_mesh/`
- Current Adapter Stack: `flutter_app/lib/features/alice/domain/alice_brain_service.dart:382`
- llama.cpp Docs: [Research needed]

## Related

^[{src_rel}]
