---
title: CODEBASE_INVENTORY
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CODEBASE_INVENTORY.md"]
updated: 2026-07-24
---

# Codebase Inventory

## Repo structure (top-level)

- `flutter_app/`: Flutter mobile app (Dart) with iOS/Android subprojects, assets, and platform-specific code.
- `app/`: SvelteKit web app with routes, stores, and services.
- `supabase/`: Supabase configuration, SQL migrations, and Edge Functions.
- `ios/` + `android/`: Additional native app/plugin code (not the Flutter subprojects).
- `api/`, `src/`, `services/`, etc.: Other backends/tools not covered in this audit scope.

## Flutter (Dart)

### Entry points

- `flutter_app/lib/main.dart`: main entry, Supabase initialization, deep link handling, Isar setup, and app root widget.
- `flutter_app/lib/core/background/nightly_federated_worker.dart`: Workmanager background entry (`callbackDispatcher`).

### Routing map

- `flutter_app/lib/routes/app_router.dart`: Auth-gated router.
  - Unauthenticated → `LoginScreen`.
  - Authenticated → `_PostAuthRouter` → `AiBootstrapScreen` (if AI bootstrap needed) or `HomeScreen`.

### State/DI wiring hubs

- No explicit DI framework (e.g., get_it) found in routing hubs.
- Uses `SharedPreferences`, `Isar`, and `Workmanager` for state/storage/async work.

### Services/modules (high-level)

- `features/alice/`: AI bootstrap, chat, assets, guardrails, LoRA training, and federated sync.
- `features/intensity/`: On-device intensity/strain services, dashboards, and storage.
- `features/nutrition/`: Nutrition targets and UI modules.
- `features/exercises/`: Exercise/nutrition DB download and on-device exercise DB handling.
- `features/marketplace/`: Trainer marketplace upload + CSV import.
- `features/chat/`: Trainer-client chat UI + report aggregation.
- `features/wearable/`: Watch/wearable sync services.

### Integrations

- **Supabase**: auth + storage (exercises/nutrition DBs, model downloads).
- **Background jobs**: Workmanager + `NativeScheduler`.
- **Local storage**: Isar (intensity, safety events), SQLite (exercise DB), SharedPreferences.
- **Deep links**: AppLinks and manual MethodChannel fallback for OAuth redirect handling.

### Feature flags / toggles

- `NIGHTLY_SCHEDULER_ENABLED`, `NIGHTLY_SCHEDULER_REQUIRE_CHARGING`, `NIGHTLY_SCHEDULER_REQUIRE_BATTERY_NOT_LOW`, `NIGHTLY_SCHEDULER_REQUIRE_UNMETERED_NETWORK` (`bool.fromEnvironment`).
- `ISAR_INSPECTOR` flag in `main.dart` (`bool.fromEnvironment`).

## Supabase

### Config

- `supabase/config.toml` defines storage buckets: `learning-deltas` (private), `ai-models` (public), `tts-models` (public).
- Auth configuration includes additional redirect URL for `com.evo.evotraining://auth-callback`.

### Migrations (ordered)

- `001_create_model_versions.sql`
- `002_create_canary_results.sql`
- `003_create_trainer_data.sql`
- `004_create_rls_policies.sql`
- `005_create_guardrail_autonomy_schema.sql`
- `006_create_exercises_schema.sql`
- `007_add_exercise_rls_policies.sql`
- `007_add_exercise_rls_policies_FIXED.sql`
- `008_create_all_missing_tables_and_rls.sql`
- `009_add_users_insert_policy.sql`
- `010_auto_create_user_profile.sql`
- `011_add_model_storage_path.sql`
- `012_create_nutrition_and_trainer_settings.sql`
- `013_create_trainer_tiers_and_billing.sql`
- `014_create_training_programs_and_video_workouts.sql`
- `20251126183441_fix_users_rls_self_insert.sql`
- `20251126200000_fix_users_rls_recursion.sql`
- `20251126200001_fix_users_rls_function.sql`
- `20251127000000_fix_users_rls_insert.sql`
- `20251127000001_fix_users_admin_policy.sql`
- `20251127000002_fix_users_rls_final.sql`
- `20251127000003_fix_users_rls_simple.sql`
- `20251127000004_fix_users_rls_complete.sql`
- `20251127000005_fix_users_rls_isolated.sql`
- `20251201110000_add_user_profile_units.sql`
- `20251203175841_add_pro_status_to_users.sql`
- `20251207123800_create_federated_delta_tables.sql`
- `20251216153000_create_chat_messages.sql`
- `20251216161000_create_chat_public_keys.sql`
- `20251217100000_preselection_trainer_chat.sql`
- `20251217130000_fix_users_trainers_view_clients_rls.sql`
- `20251218095000_make_encrypted_delta_records_id_text.sql`
- `20251218121000_add_federated_anti_replay.sql`
- `20260130190000_add_federated_rls_policies.sql`

### Tables (inferred from migrations)

- **Model + canary**: `model_versions`, `canary_test_results`.
- **Trainer attribution + approvals**: `trainer_attributions`, `trainer_approvals`.
- **Guardrails/autonomy**: `guardrail_versions`, `autonomy_policies`.
- **Exercise & equipment**: `exercise_database`, `equipment_recommendations`, `user_equipment_preferences`.
- **Users/workouts**: `users`, `workout_sessions`, `user_weekly_workouts`, `user_daily_nutrition`, `user_achievements_summary`.
- **Marketplace/video**: `training_programs`, `video_workout_assets`, `user_program_purchases`.
- **Federated learning**: `merge_batches`, `encrypted_delta_records`, `global_patch_metadata`, `federated_instance_counters`.
- **Chat**: `chat_messages`, `chat_public_keys`, `trainer_chat_requests`.

### RLS policies

- Defined across multiple migrations (e.g., `001`, `002`, `003`, `008`, `014`, and follow-up fixes in 2025/2026).

### Edge Functions

- `alice-sync/`
- `get-athlete-approvals/`
- `get-trainer-metrics/`
- `queue-trainer-approval/`
- `verify-trainer/`

## Svelte (SvelteKit)

### Entry points

- SvelteKit routes in `app/src/routes/`.
- `app/src/routes/+layout.server.ts`: route gating/redirects.

### Routes/pages

- `/` (server load in `+page.server.ts`)
- `/library` (client-rendered, no SSR/prerender)
- `/marketplace` (static page with tabs)
- `/marketplace/upload` (client-rendered upload page)
- `/marketplace/[programId]` (dynamic)
- `/workout/[workoutId]` (dynamic)
- `/.well-known/apple-app-site-association` (server endpoint)

### Stores

- Auth and user state: `app/src/lib/stores/auth.ts`.
- Additional stores for Alice, logs, wearable integrations, theme, etc. (`app/src/lib/stores/`).

### API clients/services

- Supabase client and API helper wrapper in `app/src/lib/supabase.ts`.
- Supabase helpers duplicated in `app/src/lib/api/supabase.ts`.
- ML services and storage access in `app/src/lib/services/ml/*`.
- Marketplace/workout services in `app/src/lib/services/*`.

### Feature flags / toggles

- `app/src/lib/services/featureFlags.ts` defines default flags and env overrides (manifest source, nightly FL, weekly export, staging upload, etc.).

## Native (iOS/Android)

### Flutter iOS

- `flutter_app/ios/Runner/AppDelegate.swift`: MethodChannels for shared model store, Alice brain, crash logs, wearable sync; background task registration.
- `flutter_app/ios/Runner/*`: Alice inference + training controller, TTS plugins, watch support.
- `flutter_app/ios/WatchApp/*`: watch app UI + models.

### Flutter Android

- `flutter_app/android/app/src/main/AndroidManifest.xml`: OAuth deep link intent filter + permissions.
- `flutter_app/android/app/src/main/java/com/gitfit/BodyScanParser.kt`: JSON sidecar parsing for body scan metrics.

### Additional native code (non-Flutter)

- `ios/App/` and `android/app/` include separate native Llama plugins and platform integration (appears distinct from Flutter mobile app).

## Integrations summary

- **Auth**: Supabase in Flutter + SvelteKit.
- **Email**: Supabase auth email (config in `supabase/config.toml`).
- **File storage**: Supabase Storage buckets (`ai-models`, `learning-deltas`, `tts-models`), plus R2 references in marketplace tables.
- **Background jobs**: Workmanager, native scheduler, iOS BGTaskScheduler.
- **Analytics**: Service stubs exist under `app/src/lib/services/analyticsService.ts` (usage not confirmed).

## Related
