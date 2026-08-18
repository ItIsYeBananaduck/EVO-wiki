---
title: TRAINER_AUDIT_DESKTOP_VIABILITY
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/TRAINER_AUDIT_DESKTOP_VIABILITY.md"]
updated: 2026-07-24
---

# Trainer Side Audit & Desktop Viability Assessment

**Date**: 2026-02-14
**Scope**: Every trainer-related file, service, DB table, edge function, and UI screen across the entire codebase.

---

## 1. Complete Trainer File Inventory

### Flutter App — Presentation (UI Screens)

| File                                  | Lines               | Maturity              | Notes                                                                                                                                                                                   |
| ------------------------------------- | ------------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `trainer_dashboard.dart`              | 609                 | **Functional, rough** | Live/All Clients/Revenue tabs. Revenue tab is hardcoded (`$560`, `$500`). Has debug "Create Test Session" button in production UI. Supabase polling every 5s.                           |
| `trainer_live_workout_view.dart`      | 788                 | **Most complete**     | Real-time client workout monitoring via LAN (WebSocket/mDNS) with Supabase fallback. Alice avatar with hologram beam, intensity gauge, exercise list, pose skeleton link. Chat overlay. |
| `trainer_mode_toggle.dart`            | 60                  | **Done**              | Simple toggle widget. Gates on `isTrainer` or `isRootAdmin`.                                                                                                                            |
| `trainer_settings.dart`               | 239                 | **Scaffold only**     | Notifications, theme, Alice volume — all local state, nothing persisted. Logout button is a no-op. Profile edit is a no-op.                                                             |
| `trainer_marketplace_upload.dart`     | 253                 | **Scaffold only**     | Upload form with title/description/price/category. `_handleUpload()` is `Future.delayed(2s)` — no actual upload.                                                                        |
| `trainer_workout_creator.dart`        | 322                 | **Scaffold only**     | Create/Assign/CSV Upload tabs. All are static UI shells. Assign tab has hardcoded "Sarah Johnson" and "Mike Chen". No functional logic.                                                 |
| `home_screen.dart` (trainer sections) | ~200 lines of 1300+ | **Integrated**        | Trainer mode body with 4-tab IndexedStack, trainer presence polling (6s), active client overlay, trainer nav bar. Well-integrated but tightly coupled to the main home screen.          |

### Flutter App — Domain (Services & Models)

| File                                    | Lines | Maturity     | Notes                                                                                                                                              |
| --------------------------------------- | ----- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `trainer_approval_service.dart`         | 137   | **Complete** | Thin Supabase Edge Function client for `queue-trainer-approval` and `get-athlete-approvals`. Clean API.                                            |
| `trainer_report_ingestion_service.dart` | 487   | **Complete** | Ingests weekly reports → `TrainerReportTrainingSample` for T LoRA training. SharedPreferences storage, 200-sample cap, anonymization, idempotency. |
| `trainer_report_aggregator.dart`        | 355   | **Complete** | Aggregates T samples → `TrainerAggregatedDelta` for GT LoRA upload. Mean feature vectors, statistical distributions, federated upload queue.       |
| `trainer_client_chat_screen.dart`       | 462   | **Complete** | E2E encrypted chat (Supabase + P2P WebRTC fallback). Weekly report auto-send on chat open. Trainer view auto-ingests reports for LoRA.             |
| `weekly_report_scheduler.dart`          | 201   | **Complete** | Idempotent weekly report generation and delivery. Monday-noon gate, SharedPreferences tracking.                                                    |
| `weekly_report_models.dart`             | ~100  | **Complete** | `WeeklyReportPayload`, `WeeklyReportMessage`, exercise recap, intensity, flags.                                                                    |
| `weekly_report_generator.dart`          | ~150  | **Complete** | Generates reports from on-device workout logs.                                                                                                     |
| `alice_autonomy_service.dart`           | 201   | **Complete** | Resolves autonomy mode (observe/suggest/co-author/auto) based on tier + trainer relationship. Supabase-backed with 1hr cache.                      |
| `marketplace_plans_api_service.dart`    | 86    | **Complete** | Fetches OTP plans from API. `MarketplacePlanDto` with trainerId, price, CSV URL.                                                                   |
| `app_user.dart`                         | 103   | **Shared**   | `isTrainer`, `hasTrainer`, `trainerId` fields. Shared between athlete and trainer.                                                                 |

### Flutter App — LAN Networking

| File                      | Lines | Role                                                               |
| ------------------------- | ----- | ------------------------------------------------------------------ |
| `lan_workout_server.dart` | 163   | Runs on CLIENT device. WebSocket server + mDNS advertisement.      |
| `lan_workout_client.dart` | ~150  | Runs on TRAINER device. mDNS discovery + WebSocket connection.     |
| `lan_workout_models.dart` | ~80   | `LanWorkoutSnapshot`, `DiscoveredWorkout`, connection state enums. |

### Flutter App — LoRA / ML Pipeline (Trainer-specific)

| File                                    | Lines | Role                                           |
| --------------------------------------- | ----- | ---------------------------------------------- |
| `trainer_report_ingestion_service.dart` | 487   | T LoRA sample extraction from weekly reports   |
| `trainer_report_aggregator.dart`        | 355   | GT LoRA aggregation for federated upload       |
| `lora_adapter_manager.dart`             | ~500  | Manages T and GT adapters alongside U and GU   |
| `nightly_federated_worker.dart`         | ~200  | Orchestrates nightly T/GT aggregation + upload |

### Supabase — Edge Functions

| Function                          | Lines | Status                                                                                                                        |
| --------------------------------- | ----- | ----------------------------------------------------------------------------------------------------------------------------- |
| `verify-trainer/index.ts`         | 272   | **Partially implemented** — ACE/NASM API stubs, ISSA/ACSM/OTHER → manual review. API keys not configured.                     |
| `get-trainer-metrics/index.ts`    | 273   | **Complete** — Returns certification, pattern effectiveness, revenue share, client consents. Calls `get_trainer_metrics` RPC. |
| `queue-trainer-approval/index.ts` | 100   | **Complete** — Upserts approval requests to `trainer_approvals` table.                                                        |
| `get-athlete-approvals/index.ts`  | 84    | **Complete** — Queries approval statuses for an athlete.                                                                      |

### Supabase — Database Tables (Trainer-specific)

| Table                         | Migration | Purpose                                                               |
| ----------------------------- | --------- | --------------------------------------------------------------------- |
| `trainer_attributions`        | 003       | Certification verification, pattern effectiveness, revenue share      |
| `trainer_approvals`           | 005       | Approval queue for agentic plan changes                               |
| `autonomy_policies`           | 005       | Per-athlete autonomy mode (trainer-influenced)                        |
| `trainer_client_settings`     | 012       | Alice leash level, notes, custom program per client                   |
| `trainer_tiers`               | 013       | Subscription tiers (Starter $20, Growth $50, Scale $100, Studio $175) |
| `trainer_subscriptions`       | 013       | Active subscriptions with Stripe integration                          |
| `trainer_billing_snapshots`   | 013       | Billing audit trail with Pro-user credit system                       |
| `trainer_enforcement_actions` | 013       | Upgrade warnings, suspensions                                         |
| `users.trainer_id`            | 012       | FK linking client → trainer                                           |

### SvelteKit Web App (`app/src/`)

| File                                          | Purpose                                 | Status                                 |
| --------------------------------------------- | --------------------------------------- | -------------------------------------- |
| `routes/marketplace/+page.svelte`             | Marketplace with Programs/Trainers tabs | **Functional** — fetches from Supabase |
| `routes/marketplace/[programId]/+page.svelte` | Program detail page                     | Basic                                  |
| `routes/marketplace/upload/+page.svelte`      | Upload page                             | Minimal                                |
| `lib/types/auth.ts`                           | `TrainerProfile` type definition        | Complete                               |

### Implementation Specs & Docs

| File                                    | Content                             |
| --------------------------------------- | ----------------------------------- |
| `implementation/business-model.md`      | Trainer tier pricing, credit system |
| `specs/` (multiple dirs)                | Various trainer-related specs       |
| `docs/PHASE_C_TRAINER_FLOW_COMPLETE.md` | Phase C completion summary          |

---

## 2. Maturity Assessment

### What's Actually Working

- **Trainer ↔ Client data model** — `users.trainer_id` FK, RLS policies for cross-access
- **Trainer approval queue** — Full round-trip: Flutter → Edge Function → DB → status query
- **Weekly report pipeline** — Generation → encrypted delivery → trainer ingestion → LoRA training samples
- **LAN live workout monitoring** — mDNS discovery, WebSocket streaming, Supabase fallback
- **Autonomy policy resolution** — Tier-based + trainer-influenced mode selection
- **Trainer dashboard** — Live client grid, all-clients list (from Supabase)
- **Encrypted trainer-client chat** — E2E with P2P WebRTC fallback
- **Billing schema** — Full tier/subscription/billing/enforcement DB schema with Stripe placeholders

### What's Scaffolded But Non-Functional

- **Revenue tab** — Hardcoded `$560`/`$500`, no real data
- **Trainer settings** — All toggles are local state, nothing persists
- **Marketplace upload** — `Future.delayed(2s)` fake upload
- **Workout creator** — Static UI shells, no exercise builder logic
- **Workout assignment** — Hardcoded client names, no assignment logic
- **CSV upload (trainer)** — Browse button exists, no file handling
- **Certification verification** — API keys not configured, all fall to "pending"
- **Stripe billing** — Schema exists, no Stripe integration code

### What's Missing Entirely

- **Client onboarding from trainer side** — No invite flow, no client self-registration with trainer code
- **Program builder** — No exercise selection, set/rep configuration, periodization tools
- **Client progress dashboard** — No historical charts, compliance tracking, body comp trends
- **Messaging notifications** — No push notifications for new messages or approvals
- **Trainer profile/bio editing** — No public profile page
- **Multi-trainer support** — Schema supports it but no UI for transferring clients
- **Reporting/analytics** — No aggregate client analytics, no export

---

## 3. Coupling Analysis: Trainer vs Athlete

### Tightly Coupled (Shared Code)

| Component                        | Why It's Coupled                                                     |
| -------------------------------- | -------------------------------------------------------------------- |
| `AppUser`                        | Single model with `isTrainer`/`isClient` role check                  |
| `home_screen.dart`               | Trainer mode is an `IndexedStack` overlay on the athlete home screen |
| `alice_brain_service.dart`       | Trainer context passed to Alice inference                            |
| `plans_store.dart`               | Trainer-assigned plans stored alongside self-created plans           |
| `alice_mesocycle_service.dart`   | Trainer approval checks woven into mesocycle transitions             |
| `evolora_mesh/availability.dart` | T/GT LoRA availability checks alongside U/GU                         |

### Loosely Coupled (Clean Boundaries)

| Component                               | Why It's Clean                                                 |
| --------------------------------------- | -------------------------------------------------------------- |
| `trainer_approval_service.dart`         | Standalone Supabase client, no athlete UI imports              |
| `trainer_report_ingestion_service.dart` | Only depends on `weekly_report_models` and `SharedPreferences` |
| `trainer_report_aggregator.dart`        | Only depends on ingestion service and federated uploader       |
| `trainer_client_chat_screen.dart`       | Self-contained screen with own chat sync                       |
| `lan_workout_server/client`             | Clean server/client split, shared models only                  |
| All Supabase Edge Functions             | Completely independent, HTTP-only                              |
| All DB migrations                       | Independent SQL, RLS-isolated                                  |

### Coupling Verdict

**~60% of trainer code is cleanly separable.** The main coupling points are:

1. `home_screen.dart` — trainer mode is deeply embedded (not a separate route)
2. `AppUser` — shared model (but trivially extractable)
3. LoRA pipeline — T/GT adapters share infrastructure with U/GU
4. Alice autonomy — trainer influence woven into inference decisions

---

## 4. Desktop Viability Assessment

### The Core Question

_Should the trainer experience be extracted into a standalone desktop app?_

### What a Trainer Desktop App Would Need

| Feature                       | Mobile Status              | Desktop Fit                                     | Effort                                  |
| ----------------------------- | -------------------------- | ----------------------------------------------- | --------------------------------------- |
| Client dashboard (live + all) | ✅ Working                 | ⭐ Excellent — larger screen, multi-client view | Low (port existing)                     |
| Live workout monitoring       | ✅ Working (LAN + cloud)   | ⭐ Excellent — side-by-side multi-client        | Medium (LAN discovery works on desktop) |
| Trainer-client chat           | ✅ Working (E2E encrypted) | ⭐ Excellent — keyboard-first                   | Low (port existing)                     |
| Weekly report viewing         | ✅ Working                 | ⭐ Excellent — more space for charts            | Low                                     |
| Program builder               | ❌ Scaffold only           | ⭐ Excellent — drag-and-drop, spreadsheet-like  | **High** (needs building regardless)    |
| CSV import/export             | ❌ Scaffold only           | ⭐ Excellent — file system access               | Medium                                  |
| Client analytics              | ❌ Missing                 | ⭐ Excellent — charts, tables, export           | **High** (needs building regardless)    |
| Revenue/billing dashboard     | ❌ Hardcoded               | Good — Stripe dashboard integration             | Medium                                  |
| Certification management      | ❌ Stub only               | Good                                            | Low                                     |
| Alice AI (trainer's own)      | ✅ Working                 | ⚠️ Problematic — requires on-device GGUF model  | **Blocker** (see below)                 |
| LoRA training pipeline        | ✅ Working                 | ⚠️ Problematic — llama.cpp native, Metal GPU    | **Blocker** (see below)                 |
| Marketplace upload            | ❌ Scaffold only           | Good                                            | Medium                                  |

### The Alice/LLM Blocker

The trainer's Alice runs **on-device inference** via llama.cpp with Metal GPU acceleration. This is the single biggest architectural constraint:

- **Flutter Desktop (macOS)**: Metal works. llama.cpp xcframework would need recompilation for macOS target. **Feasible but non-trivial.**
- **Flutter Desktop (Windows/Linux)**: No Metal. Would need CUDA/Vulkan llama.cpp builds. **Significant effort.**
- **Web app (SvelteKit)**: No local inference possible. Would need a cloud inference endpoint. **Architectural change.**
- **Electron/Tauri**: Same constraints as Flutter Desktop.

**However**: Most trainer-specific features **don't need on-device Alice**. The trainer's Alice is used for:

1. Coaching assistance during live monitoring (nice-to-have)
2. LoRA training from client reports (background task, could be server-side)

A desktop trainer app could function perfectly well **without on-device inference** by:

- Using a cloud API for Alice chat (OpenAI/Anthropic/self-hosted)
- Moving LoRA aggregation to a server-side pipeline
- Keeping the mobile app as the "AI-native" experience

### Recommendation

#### ✅ YES — A desktop trainer app is viable and wise, BUT...

**Not as a full extraction. As a purpose-built web app.**

#### Rationale

1. **The SvelteKit app already exists** (`app/src/`) with marketplace pages, auth types, and Supabase integration. It's the natural foundation.

2. **Trainers work at desks.** Program building, client analytics, CSV management, billing review — these are all keyboard-and-mouse workflows. Mobile is awkward for these.

3. **The coupling is manageable.** 60% of trainer code is already cleanly separated. The tightly coupled parts (home_screen, AppUser, LoRA) don't need to come to desktop.

4. **You don't need on-device Alice for desktop.** The trainer desktop app should be a **management tool**, not an AI companion. Alice on mobile is the differentiator for athletes. Trainers need dashboards, not chatbots.

5. **The scaffolded features need building anyway.** Program builder, analytics, billing — these are all better built desktop-first and then ported to mobile, not the other way around.

#### Recommended Architecture

```
┌─────────────────────────────────────┐
│     Trainer Desktop (SvelteKit)     │
│                                     │
│  • Client management dashboard      │
│  • Program builder (drag-and-drop)  │
│  • Analytics & reporting            │
│  • Billing & revenue (Stripe)       │
│  • Marketplace management           │
│  • Chat (via Supabase Realtime)     │
│  • Live workout viewer (WebSocket)  │
│  • CSV import/export                │
│                                     │
│  Auth: Supabase (shared)            │
│  Data: Same Supabase DB (shared)    │
│  No local AI inference needed       │
└─────────────────────────────────────┘
           │
           │ Supabase (shared DB + auth + Edge Functions)
           │
┌─────────────────────────────────────┐
│     Athlete Mobile (Flutter)        │
│                                     │
│  • On-device Alice AI               │
│  • Live workout tracking            │
│  • Nutrition logging                │
│  • LAN broadcast to trainer         │
│  • Weekly report generation         │
│  • LoRA training pipeline           │
└─────────────────────────────────────┘
```

#### What to Build First (Priority Order)

1. **Client management dashboard** — Port `trainer_dashboard.dart` logic to SvelteKit. Real data, not hardcoded.
2. **Program builder** — The highest-value missing feature. Drag-and-drop exercise selection, set/rep config, mesocycle planning. Desktop-first.
3. **Live workout viewer** — WebSocket connection to client's LAN server or Supabase realtime. Multi-client grid view.
4. **Chat** — Supabase Realtime + existing E2E encryption. Weekly report rendering.
5. **Analytics** — Client progress charts, compliance tracking, body comp trends.
6. **Billing** — Stripe integration against existing DB schema.

#### What to Keep Mobile-Only

- On-device Alice AI inference
- LoRA training pipeline (T/GT)
- LAN workout server (runs on client phone)
- Pose estimation / camera features
- Apple Music integration

---

## 5. Technical Debt & Issues Found

| Issue                                                   | Severity | Location                                             |
| ------------------------------------------------------- | -------- | ---------------------------------------------------- |
| Revenue tab hardcoded `$560`/`$500`                     | P2       | `trainer_dashboard.dart:539-550`                     |
| Debug "Create Test Session" button in production        | P1       | `trainer_dashboard.dart:199-204`                     |
| Trainer settings not persisted                          | P2       | `trainer_settings.dart` (all local state)            |
| Marketplace upload is fake                              | P2       | `trainer_marketplace_upload.dart:238-239`            |
| Workout creator is entirely static                      | P2       | `trainer_workout_creator.dart`                       |
| Assign tab has hardcoded client names                   | P3       | `trainer_workout_creator.dart:184-194`               |
| `_sqrt()` reimplemented instead of `import 'dart:math'` | P3       | `trainer_report_aggregator.dart:261-268`             |
| Certification API keys never configured                 | P2       | `verify-trainer/index.ts`                            |
| No Stripe integration code exists                       | P2       | Schema only in migration 013                         |
| Trainer presence polling at 5s + 6s (two timers)        | P3       | `trainer_dashboard.dart:31` + `home_screen.dart:646` |

---

## 6. Summary

| Dimension                       | Score    | Notes                                                                                                                                                        |
| ------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Data model completeness**     | 8/10     | DB schema is thorough. Missing: program templates table, client progress snapshots.                                                                          |
| **Backend completeness**        | 7/10     | Edge functions work. Missing: Stripe webhooks, notification service.                                                                                         |
| **Mobile UI completeness**      | 4/10     | Dashboard and live view work. Everything else is scaffolded.                                                                                                 |
| **Business logic completeness** | 7/10     | Approval queue, weekly reports, LoRA pipeline all solid. Missing: billing logic, program assignment.                                                         |
| **Desktop viability**           | **9/10** | Excellent candidate. Most value-add features are desktop-native workflows. SvelteKit foundation exists. Shared Supabase backend eliminates data sync issues. |
| **Desktop wisdom**              | **8/10** | Wise move. Trainers need management tools, not mobile AI. Build desktop-first for the features that are currently scaffolded.                                |

**Bottom line**: The trainer side has a solid backend foundation and a few working mobile features, but most of the UI is scaffolded shells. A desktop web app is not just viable — it's the _right_ platform for the majority of trainer workflows. Build it on the existing SvelteKit app with the shared Supabase backend.

## Related

^[{src_rel}]
