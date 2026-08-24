---
title: EVOLoRA_Mesh_Compliance_Report
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOLoRA_Mesh_Compliance_Report.md"]
updated: 2026-07-24
---

# EVOLoRA Mesh v1 Compliance Report (Repo Audit)

**Scope**

- Repo root: `git-fit-fresh/`
- Code inspected: Flutter (`flutter_app/`), SvelteKit/Capacitor (`app/`), Supabase (`supabase/`), federated server (`federated-server/`), VectorRAG module (`VectorRAG/`).

**Method**

- This report marks each requirement as:
  - ✅ **compliant** (explicitly implemented; citations provided)
  - ⚠️ **partially compliant** (implemented but missing required constraints/coverage)
  - ❌ **non-compliant** (missing, ambiguous, or contradicted by code)
- If behavior is only implied by comments/specs and not enforced by code, it is ❌.

---

## Architecture map (what exists today)

### Roles & permissions

- **User roles** are represented in Supabase `public.users.role` with allowed values `client | trainer | admin`.
  - **Evidence**: `supabase/migrations/008_create_all_missing_tables_and_rls.sql` `CREATE TABLE IF NOT EXISTS users (... role text DEFAULT 'client' CHECK (role IN ('client', 'trainer', 'admin')) ...)`.
- **Trainer relationship** is represented by `users.trainer_id` (referenced in chat RLS and user-selection policies).
  - **Evidence**: `supabase/migrations/20251216153000_create_chat_messages.sql` uses `users.trainer_id` in `chat_messages_select/insert`.
  - **Evidence**: `supabase/migrations/20251217130000_fix_users_trainers_view_clients_rls.sql` policy predicate `trainer_id = auth.uid()`.
- **Flutter user model** includes `role`, `trainerId`, and derived `hasTrainer`.
  - **Evidence**: `flutter_app/lib/features/auth/domain/app_user.dart`.

### Trainer approvals

- A **trainer approval queue** exists in Supabase.
  - **Evidence**: `supabase/migrations/005_create_guardrail_autonomy_schema.sql` `CREATE TABLE IF NOT EXISTS trainer_approvals` with RLS restricted to `service_role`.
- Two Edge Functions provide write/read access:
  - Queue: `supabase/functions/queue-trainer-approval/index.ts`.
  - Fetch athlete approvals: `supabase/functions/get-athlete-approvals/index.ts`.
- Flutter integrates approvals for **mesocycle progression** specifically.
  - **Evidence**: `flutter_app/lib/features/alice/domain/alice_mesocycle_service.dart` uses `trainerApprovalService.queueApproval(...)` and `reconcileMesocycleApprovalsForUser`.

### Plans & mesocycles

- Flutter stores training plans locally in SharedPreferences, using `TrainingPlanRecord` with fields:
  - `id`, `name`, `source`, `isActive`, `cycleLengthWeeks`, `workoutPlan`, and trainer-approval metadata (`trainerApprovalStatus`, `pendingActionHash`, `pendingProposedPlan`).
  - **Evidence**: `flutter_app/lib/features/alice/domain/plans_store.dart`.
- There is **no** immutable PlanArtifact/version model implemented (see compliance matrix).

### Encrypted chat (transport)

- Flutter implements E2E-style encryption (X25519 + AES-GCM) and stores encrypted messages in Supabase.
  - **Evidence**: `flutter_app/lib/features/chat/domain/chat_crypto.dart`.
  - **Evidence**: `flutter_app/lib/features/chat/domain/chat_key_manager.dart` stores keys in `flutter_secure_storage` and publishes public keys to `chat_public_keys`.
  - **Evidence**: `flutter_app/lib/features/chat/domain/chat_sync_service.dart` inserts encrypted messages into Supabase `chat_messages`.
  - **Evidence**: `supabase/migrations/20251216153000_create_chat_messages.sql` schema + RLS for client↔trainer/admin.

### Vector RAG (on-device)

- A standalone `VectorRAG/` module exists implementing:
  - 30-day local raw log storage and prune.
    - **Evidence**: `VectorRAG/localStore.ts` constant `MAX_DAYS = 30` and `pruneOlderThan()` deleting dated JSON.
  - Nightly vector generation and encryption of `{ date, logs }` JSON into `.bin` artifacts.
    - **Evidence**: `VectorRAG/VectorRAG.ts` method `queueNightlyVectorGeneration()` + `buildArtifact()` with AES-GCM.
  - Encrypted upload queue.
    - **Evidence**: `VectorRAG/uploadQueue.ts`.
- The Svelte app’s `vectorRagService` is currently a **no-op stub**.
  - **Evidence**: `app/src/lib/services/vectorRag/index.ts` comment and exported `recordWorkout` no-op.

### Federated learning (ACIF)

- Svelte/Capacitor code includes QLoRA training + encrypted delta upload/queueing.
  - **Evidence**: `app/src/lib/services/ml/qloraTrainer.ts` (`runDailyTraining()` and 24h gating).
  - **Evidence**: `app/src/lib/services/ml/deltaQueue.ts` weekly upload window `isSundayMidnightWindow()` and persistence.
  - **Evidence**: `app/src/lib/services/privacy/deltaEncryptor.ts` AES-256-GCM encryption.
- Flutter has a federated uploader + crypto, but the delta queue is explicitly described as a stub.
  - **Evidence**: `flutter_app/lib/features/alice/domain/federated_delta_queue.dart` docstring lines ~224–228 “Stub implementation … does not yet implement … encryption” (note: file also contains key derivation/encryption helpers; the comment indicates incomplete port).
- Federated server accepts encrypted deltas and stores metadata.
  - **Evidence**: `federated-server/src/api/upload_delta.py` stores into Supabase Storage bucket `learning-deltas` and `encrypted_delta_records`.
  - **Evidence**: `supabase/migrations/20251207123800_create_federated_delta_tables.sql`.

---

## Compliance matrix

### A) Roles & Knowledge-Only LoRAs

| Requirement                                     | Status | Evidence / Notes                                                                                                                                                                                                                                                                                             |
| ----------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Trainer accounts MUST have LoRAs: U, T, GU, GT  | ❌     | No code establishes four adapters per trainer. Existing code references **only** `userLoRA` and a single `globalPatch` (see `app/src/lib/services/ml/modelUpdater.ts` constants like `USER_LORA_FILENAME`, `GLOBAL_PATCH_FILENAME` from earlier repo search). No `T` (trainer adapter) artifacts identified. |
| User accounts MUST have: U, GU, GT              | ❌     | Same as above; no separate GU/GT adapters exist in code.                                                                                                                                                                                                                                                     |
| No tone/personality LoRA in beta                | ❌     | No enforcement exists; there is no adapter classification layer or prohibition mechanism in code.                                                                                                                                                                                                            |
| All LoRAs are knowledge-only                    | ❌     | No enforcement exists; training pipeline does not tag adapters as “knowledge-only”.                                                                                                                                                                                                                          |
| If user has assigned trainer → user-side GT = 0 | ❌     | No GT exists; no gating logic implemented. Trainer assignment exists (`users.trainer_id`), but there is no GT weighting/gating system.                                                                                                                                                                       |

### B) Context Routing & Weight=0 Gating

| Requirement                                                                    | Status | Evidence / Notes                                                                                                                    |
| ------------------------------------------------------------------------------ | ------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| Explicit LoRA router with contexts listed (PLAN_DRAFT_MAJOR, … PATTERN_MINING) | ❌     | Searches across `app/src` and `flutter_app/lib` found **no occurrences** of required context identifiers. No router module present. |
| Non-trainer accounts → T = 0                                                   | ❌     | No `T` adapter exists; no gating logic.                                                                                             |
| Assigned trainer on user → GT = 0                                              | ❌     | No `GT` adapter exists; no gating logic.                                                                                            |
| Trainer plan authoring → trainer U = 0 unless explicit self-mode               | ❌     | No router/gating exists.                                                                                                            |
| Major plan logic: T dominant identity, GT>GU priors                            | ❌     | No routing/weight code exists.                                                                                                      |

### C) Trainer Plan Pipeline — Propose → Approve → Publish

| Requirement                                                   | Status | Evidence / Notes                                                                                                                                                                                                   |
| ------------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Alice may draft plans on trainer side                         | ❌     | No trainer-side plan authoring implementation found. Flutter stores local plans for the user (`TrainingPlanStore`), but no trainer UI pipeline enforcing propose/approve/publish.                                  |
| Trainer must approve before publishing                        | ⚠️     | There is an approval mechanism (`trainer_approvals` + Edge Functions). However Flutter uses it only for `mesocycle_progression` in `alice_mesocycle_service.dart`. No generalized “publish gate” exists for plans. |
| Trainer edits are logged and trained on                       | ❌     | No trainer edit log pipeline tied to training (no dataset builder or adapter-specific training for trainer edits).                                                                                                 |
| Novel element detection + explicit accept/edit/reject logging | ❌     | No novelty detection module or per-element accept/reject logging is present.                                                                                                                                       |
| No silent novelty. Ever.                                      | ❌     | Not enforced; plan generation/progression can modify plan locally when `hasTrainer == false` in `alice_mesocycle_service.dart` (direct overwrite of `workoutPlan`).                                                |

### D) Versioning — Immutable Intended (Option A)

| Requirement                                                                                       | Status | Evidence / Notes                                                                                                                                                                                                                          |
| ------------------------------------------------------------------------------------------------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PlanArtifact has stable `plan_id`, monotonic `version`, canonical JSON `hash`, immutable versions | ❌     | No PlanArtifact table or model exists. Flutter’s `TrainingPlanRecord` has only `id` and embeds a mutable `workoutPlan` JSON. No version or hash fields exist in `TrainingPlanRecord` (`plans_store.dart`).                                |
| WorkoutLog includes `plan_id`, `plan_version`, `day_id`                                           | ❌     | Flutter `OnDeviceWorkoutLog` has no plan linkage fields (`flutter_app/lib/features/intensity/domain/intensity_models.dart`). Supabase `workout_sessions` table also lacks plan_id/version in `008_create_all_missing_tables_and_rls.sql`. |
| Intended plan reconstructed from logs                                                             | ❌     | No implementation; missing required fields.                                                                                                                                                                                               |

### E) Update Schedule Rule

| Requirement                                                   | Status | Evidence / Notes                                                                                                                                                                                               |
| ------------------------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Normal updates only at mesocycle boundaries                   | ⚠️     | `alice_mesocycle_service.dart` applies progression when the calendar week increments (boundary-like). However, no PlanArtifact versioning is created and no global enforcement prevents other mid-cycle edits. |
| Mid-mesocycle updates only for safety, still new plan version | ❌     | No safety-only override pipeline with version creation exists.                                                                                                                                                 |
| No plan changes apply mid-set                                 | ❌     | No enforcement exists at workout execution level; live workout can update rest/sets dynamically in UI, but there is no guard that prohibits plan mutation mid-set.                                             |

### F) User Execution Logging — Intended vs Done

| Requirement                                                                                        | Status | Evidence / Notes                                                                                                                                                                  |
| -------------------------------------------------------------------------------------------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| On-device log executed sets/reps/load/rest                                                         | ✅     | `OnDeviceWorkoutSet` includes `reps`, `weight`, `restSeconds`, etc. (`flutter_app/lib/features/intensity/domain/intensity_models.dart`).                                          |
| Compute divergence vs intended                                                                     | ❌     | No divergence computation exists; `OnDeviceWorkoutLog` has no link to intended plan and no divergence model/table.                                                                |
| Store divergence on day first occurs                                                               | ❌     | Not implemented.                                                                                                                                                                  |
| Capture reason chips mandatory over thresholds                                                     | ❌     | Not implemented.                                                                                                                                                                  |
| User-side Alice may adjust sets/reps/load/rest only; no exercise swaps mid-mesocycle except safety | ❌     | No policy enforcement exists; live workout allows adding exercises (e.g., `LiveWorkoutScreen._addExercise()` in `live_workout_screen.dart`). Safety-only constraints not encoded. |

### G) Weekly Reports (via Encrypted Chat)

| Requirement                                                                               | Status | Evidence / Notes                                                                                                                                                                                                            |
| ----------------------------------------------------------------------------------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Once per week user-side Alice generates report                                            | ❌     | No weekly report generator found in Flutter or app that produces a report payload intended for trainer ingestion via chat.                                                                                                  |
| Sent via encrypted chat                                                                   | ⚠️     | Encrypted chat transport exists (Flutter chat modules + Supabase `chat_messages`). But no report envelope flow exists.                                                                                                      |
| Message includes `report_id`, canonical `payload_hash`, signature or provable sender auth | ❌     | Chat message schema includes only `encrypted_content`, `iv`, `timestamp` (`supabase/migrations/20251216153000_create_chat_messages.sql`). No report metadata fields. No signature scheme for weekly reports is implemented. |
| Trainer verifies, deduplicates, stores, indexes into RAG                                  | ❌     | No trainer-side ingestion endpoint or verification/dedupe store exists. VectorRAG is on-device for workouts and is not scoped to weekly report cards.                                                                       |

### H) Trainer Pattern Mining

| Requirement                                                      | Status | Evidence / Notes                                                                                                                                              |
| ---------------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Trainer-side Alice detects patterns per user over weeks          | ❌     | No trainer-side mining pipeline exists; VectorRAG admin screen is gated to root admin and operates on raw workouts (`app/src/routes/admin/rag/+page.svelte`). |
| Detect aggregates across all trainer clients                     | ❌     | No implementation found.                                                                                                                                      |
| Aggregates bias proposal ordering but never override constraints | ❌     | No router/prior system exists.                                                                                                                                |

### I) RAG Scope & Retention (Trainer Device)

| Requirement                                                    | Status | Evidence / Notes                                                                                                                                                    |
| -------------------------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RAG stores Weekly Memory Cards: one per client-week            | ❌     | VectorRAG stores per-day artifacts with `{ date, logs }` and vectors; no “weekly memory card” schema. (`VectorRAG/VectorRAG.ts` `buildArtifact()` stores raw logs.) |
| Summaries/tags/rollups only; no raw workouts                   | ❌     | VectorRAG stores raw `logs` JSON in encrypted artifact payload (and local raw logs in `rag/logs/*.json`). (`VectorRAG/VectorRAG.ts`, `VectorRAG/localStore.ts`).    |
| Retention capped 12–24 months + optional quarterly compression | ❌     | Local store retention is 30 days only; cloud artifact retention policy not defined in code; no 12–24 month retention or compression jobs.                           |
| Retrieval filtered by client/time/tags then semantic search    | ❌     | VectorRAG query is date range + optional exercise filter; no client/tags dimension, no semantic search pipeline beyond vector similarity placeholder.               |

---

## Summary verdict

**EVOLoRA Mesh v1** is currently **❌ non-compliant**.

Key blockers:

- No multi-adapter (U/T/GU/GT) mesh implementation.
- No explicit router contexts or weight=0 gating.
- No immutable PlanArtifact versioning and no plan_id/version linkage in workout logs.
- No weekly report envelope generation/signing/dedupe pipeline.
- RAG implementation exists but does not match trainer-week memory card constraints.

---

## Immediate follow-ups (inputs needed)

To complete 100% compliance, we will need to confirm:

- Whether EVOLoRA Mesh is intended to be implemented in Flutter, in the Capacitor app, or both.
- Where trainer-side Alice runs (trainer device app? web console? server-side?).

(These do not change the compliance markings above; they only affect implementation planning.)

## Related
