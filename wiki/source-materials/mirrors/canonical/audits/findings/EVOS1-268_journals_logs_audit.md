---
title: "EVOS1-268 — Audit Journals & Logs Across Apps"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-05-02
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-268 — Audit Journals & Logs Across Apps

Date: 2026-05-02
Issue: EVOS1-268

## Scope
Audit current state of journals and logs across:
- EVOtraining
- EVOmind
- EVOconnect
- EVOlearn

## Method
- Audited app/runtime code and docs for `journal`, `log`, `history`, `store`, and persistence patterns.
- Focused on concrete persisted artifacts and explicit runtime logging constructs.

---

## 1) Inventory — Existing Logs Per App

### EVOtraining (implemented app in this repo)

**Explicit logs / log-like stores**
- `session_log_service.dart`: explicit session logging for Alice/workout-facing flows.
- `plan_deviation_log.dart`: captures plan deviations (structured behavior log).
- `chat_history_manager.dart` + chat session models/stores: conversation history persistence (journal-like behavioral memory).
- `persistent_decision_log_store.dart` (EvoLoRA mesh): decision logs for safety/overload/substitution logic.
- `crash_logger_service.dart` + diagnostics surfaces (`diag_log.dart`, debug log viewer screen): operational/runtime diagnostics.
- Federated queue/aggregation artifacts (`federated_delta_queue.dart`, `user_delta_aggregator.dart`): event-pattern logging for model-learning transport.

**Implicit logs not formally unified**
- Multiple feature-local stores act as behavioral logs without a shared schema:
  - nutrition scan/product persistence
  - training data samples and reset/transform flows
  - wearable/live workout sync states
- These are logged as feature artifacts, but not classified in one “journal + event log” domain model.

---

### EVOconnect (implemented app + shared connect/core runtime)

**Explicit logs / audit stores**
- Shared `packages/core` task runtime uses explicit `TaskLogEntry` model and `LogStore` (`connect_logs.db`).
- `task_store` keeps task lifecycle and embedded logs.
- Dedicated audit streams exist conceptually and in tests for terminal/browser/vault execution outcomes.

**Implicit logs not formally unified**
- Orchestrator and control-path lifecycle events are distributed across runtime/task records and ad hoc metadata.
- No cross-app cognition journal schema yet; logs are runtime-centric vs. user-reflection-centric.

---

### EVOmind

**Current state in this repo**
- No standalone `apps/evo_mind` implementation found.
- Mind journaling appears in product/spec language (mode=`journal`, “Mind app journal view”), but no concrete app-local journal store implementation is present in this codebase snapshot.

**Result**
- Journal/log system for EVOmind is currently **missing in implementation** here (baseline gap).

---

### EVOlearn

**Current state in this repo**
- No standalone `apps/evo_learn` implementation found.
- EVOlearn appears in role-overlay/identity and ecosystem docs, not as a concrete executable app package with journal/log storage.

**Result**
- Journal/log system for EVOlearn is currently **missing in implementation** here (baseline gap).

---

## 2) Journal-like Behavior (Cross-App)

Observed journal-like behavior exists but is fragmented:
- Chat history persistence (conversation memory) behaves like reflective journaling.
- Plan deviations and decision logs behave like structured training journals.
- Nutrition scans and behavioral deltas capture timeline events similar to activity journaling.

However, these are split by feature and app/runtime layer without a unified “Cognition Journal” abstraction.

---

## 3) Missing Journal System (Baseline Gaps)

1. **No unified cross-app journal domain model**
   - No shared schema for entry type, source app, temporal context, confidence/quality, privacy tier, embedding status.

2. **No explicit EVOmind journal implementation in-repo**
   - Specs reference Mind journaling, but code implementation is absent.

3. **No explicit EVOlearn journal implementation in-repo**
   - Identity/docs reference exists, but app + storage path absent.

4. **No cross-app ingestion router for journal entries**
   - Existing logs are app-local/runtime-local; cognition-layer ingestion contract is not yet wired.

5. **No unified retention + redaction policy at journal layer**
   - Privacy controls exist in pockets (federated/privacy docs), but no single journal retention/redaction pipeline visible.

---

## 4) Misclassified Data (Current Risk Areas)

1. **Chat history as “feature state” instead of journal artifact**
   - Conversation memory currently acts like a longitudinal journal but is classified as chat/session storage.

2. **Plan deviation / decision logs as “internal engine artifacts” only**
   - These are high-value cognition signals but not modeled as first-class journal entries.

3. **Nutrition scan/product persistence as utility cache only**
   - Some records represent meaningful user behavior chronology and may belong in journal/event ingestion.

4. **Runtime audit logs mixed with user-behavior signals**
   - Connect/core task logs are operational and should be separated from reflective/user cognition journals in future schema.

---

## 5) Baseline Summary (Acceptance Criteria Mapping)

### Full inventory of logs per app
- Completed for EVOtraining and EVOconnect from concrete implementation.
- EVOmind and EVOlearn identified as not implemented in this repo snapshot.

### Journal gaps identified
- Unified journal abstraction missing.
- Mind/Learn journal implementations missing.
- Cross-app cognition ingestion mapping missing.

### Misclassified data identified
- Chat history, plan deviation/decision logs, and some nutrition artifacts are behavior-journal candidates currently classified as feature-local state.
- Operational runtime audit logs should remain separate from user journal layer.

---

## Recommended Next Step (for EVOS1-266 parent initiative)
Create a shared `journal_entry` contract (packages/domains) with:
- `entry_id`, `user_id`, `source_app` (`training|mind|connect|learn`), `entry_kind`, `timestamp`, `content_ref`, `metadata`, `privacy_tier`, `ingestion_state`, `embedding_ref`.

Then add per-app mappers:
- EVOtraining: chat/session/deviation/decision/nutrition signals -> journal_entry.
- EVOconnect: task orchestration logs -> operational audit only (explicitly excluded from reflective journal unless tagged).
- EVOmind/EVOlearn: scaffold app-local journal stores first, then ingestion adapters.