---
title: DOCS_SYNC_PLAN
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/DOCS_SYNC_PLAN.md
updated: 2026-07-24
---

# Docs Sync Plan

## Docs that appear aligned with current code

- **SUPABASE_SETUP.md**: Matches the presence of direct Supabase usage (exercise_database, equipment tables, and RLS policies in migrations), and references Supabase env vars used by the Svelte/Flutter clients.
- **FLUTTER_SETUP.md**: Still relevant as a local Flutter bootstrap guide, though it is Windows-path-specific.

## Docs that conflict or look stale

- **README.md**: Emphasizes a Fly.io `api/` backend and a Capacitor-first Svelte setup; it doesn’t match the current Flutter-centric mobile wiring and Supabase-first backend usage.
- **OAUTH_SETUP.md**: References `/auth/login` and Vercel URLs that do not appear in the SvelteKit route list; current Flutter OAuth deep-link handling is `gitfit://auth-callback`.

## Recommended actions

- **Keep (with minor updates):**
  - `SUPABASE_SETUP.md` → add references to storage buckets (`ai-models`, `learning-deltas`, `tts-models`) and Edge Functions wiring status.
  - `FLUTTER_SETUP.md` → split into platform-agnostic steps + Windows-specific appendix.

- **Update or mark historical:**
  - `README.md` → update repo structure to highlight `flutter_app/`, `app/`, and `supabase/`; move Fly.io API notes to historical context if still used.
  - `OAUTH_SETUP.md` → reconcile with current routing (SvelteKit routes vs Flutter deep links) or mark as legacy.

- **Delete (if not used):**
  - Any docs referencing Capacitor-only flows or deprecated `/auth/login` routes, unless those routes are reintroduced.

## Proposed `CURRENT_ARCHITECTURE.md` outline

1. **High-Level Overview**
   - Flutter app + SvelteKit web + Supabase backend + native integrations.
2. **Flutter App**
   - Entry points (`main.dart`, Workmanager worker).
   - Router/auth flow.
   - Key modules (Alice AI, workouts, nutrition, wearable, marketplace).
   - Background tasks and local storage (Isar/SQLite).
3. **SvelteKit Web**
   - Routes/pages.
   - Stores (auth + user state).
   - Supabase client and feature flags.
4. **Supabase Backend**
   - Schema summary (tables + RLS).
   - Edge Functions and current wiring status.
   - Storage buckets and file flows.
5. **Native Integrations**
   - iOS AppDelegate channels.
   - Android manifest deep links + native helpers.
6. **Cross-Layer Wiring**
   - Auth + deep link flows.
   - Model/asset downloads.
   - Trainer approvals + metrics.
7. **Gaps / Pending Work**
   - Unwired routes, placeholders, feature flags.

## Related

^[source-materials/mirrors/doctrine/DOCS_SYNC_PLAN.md]
