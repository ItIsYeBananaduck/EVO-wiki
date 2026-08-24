---
title: supabase-calendar-dependency-map
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/supabase-calendar-dependency-map.md
updated: 2026-07-24
---

# Supabase Calendar Dependency Map + Removal Checklist

Produced by EVOTRA-583 Cluster A (A1–A4). No code changes — written artifact only.
This document is the execution contract for EVOTRA-588 Cluster B.

---

## A1 — Supabase Calendar Write Surface

All Supabase writes are **deferred queue** operations (not hot-path direct writes).
The local Isar store is written first; Supabase is written asynchronously via `CalendarSyncQueue`.

### Write sites

| File | Method | Table | Operation | Path |
|------|--------|-------|-----------|------|
| `flutter_app/lib/features/coach/data/calendar_sync_queue.dart` | `_syncTicket()` line 287 | `coach_availability` | `upsert` | Deferred queue drain |
| `flutter_app/lib/features/coach/data/calendar_sync_queue.dart` | `_syncTicket()` line 290 | `coach_availability` | `delete` | Deferred queue drain |
| `flutter_app/lib/features/coach/data/calendar_sync_queue.dart` | `_syncTicket()` line 294 | `coach_appointments` | `upsert` | Deferred queue drain |
| `flutter_app/lib/features/coach/data/calendar_sync_queue.dart` | `_syncTicket()` line 297 | `coach_appointments` | `delete` | Deferred queue drain |
| `flutter_app/lib/features/coach/data/calendar_sync_queue.dart` | `_syncTicket()` line 301 | `coach_scheduled_checkins` | `upsert` | Deferred queue drain |
| `flutter_app/lib/features/coach/data/calendar_sync_queue.dart` | `_syncTicket()` line 304 | `coach_scheduled_checkins` | `delete` | Deferred queue drain |

### Audit logging (DB-side only — no Flutter write)

The `audit_logs` table is written exclusively by Postgres triggers defined in:
`supabase/migrations/20260517150000_create_coach_calendar_schema.sql` (triggers on lines 213–229).

`CalendarAuditLogger` in Flutter writes **to local file only** (console + local JSONL). It does **not** write to `audit_logs` via Supabase client. No Flutter → Supabase write path for audit logs.

### Enqueue sites (where sync tickets are created)

All enqueue calls are in `CalendarRepository` methods which write to local store first, then call `_syncQueue.enqueue(...)`:

| CalendarRepository method | Operation enqueued | Notes |
|---------------------------|--------------------|-------|
| `saveAvailabilityWindow()` | `createAvailability` / `updateAvailability` | Also logs to CalendarAuditLogger |
| `saveAppointmentSlot()` | `createAppointment` / `updateAppointment` | — |
| `saveScheduledCheckIn()` | `createCheckIn` / `updateCheckIn` | — |
| `saveBlockedTime()` | `createAvailability` / `updateAvailability` | Converts BlockedTime → AvailabilityWindow for sync; also logs audit |
| `deleteAvailabilityWindow()` | `deleteAvailability` | — |
| `deleteAppointmentSlot()` | `deleteAppointment` | — |
| `deleteScheduledCheckIn()` | `deleteCheckIn` | — |
| `deleteBlockedTime()` | `deleteAvailability` | Converts to AvailabilityWindow for sync |

**Key finding**: Removing Supabase sync requires only removing `CalendarSyncQueue` and the `enqueue()` calls in `CalendarRepository`. Local Isar writes are in `CalendarLocalStore` and are entirely separate. No Isar code changes needed.

---

## A2 — Supabase Calendar Read and Realtime Surface

### Direct reads

`CalendarRepository` has **zero direct Supabase select/rpc calls**. All reads are delegated to `CalendarLocalRepository` (Isar). The sync layer is write-only from Flutter's perspective — there is no pull/fetch from Supabase into Isar.

Search result: `supabase.rpc(` — **zero matches** in `flutter_app/lib/features/coach/`.

### Realtime subscriptions

| File | Class | Channel | Table | Events | Consumer |
|------|-------|---------|-------|--------|----------|
| `flutter_app/lib/features/coach/data/calendar_realtime_subscription.dart` | `CalendarRealtimeSubscription` | `coach_appointments:{coachId}` | `coach_appointments` | INSERT, UPDATE | `AliceResponseNotificationService.notifyCalendarEvent()` |

#### Subscription behavior
- **INSERT handler** (`_onInsert`): fires `CalendarNotificationEvent.appointmentRequested` → notifies coach of new booking request.
- **UPDATE handler** (`_onUpdate`): fires `appointmentConfirmed` (to client) or `appointmentCancelled` (to coach or client) based on status transition.
- The subscription is a **read-only observer** — it never writes to Supabase or Isar.
- Subscription failure is silently swallowed (non-fatal design).

#### Connectivity detection (secondary Supabase use)
`CalendarSyncQueue._listenToConnectivity()` uses:
```dart
Supabase.instance.client.realtime.onOpen(...)
Supabase.instance.client.realtime.onClose(...)
Supabase.instance.client.realtime.isConnected
```
This is used solely to determine whether to attempt queue drain. Must be replaced with a connectivity package (e.g., `connectivity_plus`) when Supabase realtime is removed.

### Supabase migrations (schema)

| Migration file | Tables created | Status |
|----------------|---------------|--------|
| `supabase/migrations/20260517150000_create_coach_calendar_schema.sql` | `coach_availability`, `coach_appointments`, `coach_scheduled_checkins`, `audit_logs` (triggers) | Applied |
| `supabase/migrations/20260517150100_create_coach_calendar_rls.sql` | RLS policies for all 3 tables | Applied |

These migrations must be reversed by a down migration in Cluster B (B5).

---

## A3 — UI Sync Layer Dependencies

### Components consuming `CalendarSyncQueueStatus`

| Component | File | What it reads | Change required on removal |
|-----------|------|---------------|---------------------------|
| `_buildSyncStatus()` widget | `calendar_tab.dart` lines 324–386 | `_bloc.syncStatusStream` → `StreamBuilder<CalendarSyncQueueStatus>` | Remove `_buildSyncStatus()` widget; remove the `StreamBuilder` block and its `SizedBox.shrink()` idle branch |
| `CalendarBloc.syncStatusStream` | `calendar_bloc.dart` line 280 | Delegates to `_availabilityService.syncStatusStream` → `CalendarSyncQueue.statusStream` | Remove getter + all `syncNow()` / `pendingSyncCount` members |
| `CalendarBloc.pendingSyncCount` | `calendar_bloc.dart` line 284 | `_availabilityService.pendingSyncCount` | Remove |
| `CalendarBloc.syncNow()` | `calendar_bloc.dart` line 287–289 | `_availabilityService.syncNow()` | Remove |
| `CalendarTab._initBloc()` | `calendar_tab.dart` line 100 | `calendarRepo.initialize()` — starts sync queue | Remove `calendarRepo.initialize()` call; `CalendarRepository` constructor no longer needs `syncQueue` |

### `CalendarRepository` field dependency

| Component | File | Field | Change required |
|-----------|------|-------|-----------------|
| `CoachClientWorkspace` | `flutter_app/lib/features/home/presentation/coach_client_workspace.dart` | `CalendarRepository? calendarRepository` (optional) | Field itself is fine to keep; it holds the local repo. Just must not inject `CalendarSyncQueue` into it. Constructor simplifies. |
| `ClientBookingView` | `flutter_app/lib/features/coach/presentation/client_booking_view.dart` | `CalendarRepository calendarRepository` | No change — still uses local repo API unchanged |
| `_CheckInsSection` | `coach_client_workspace.dart` | `CalendarRepository calendarRepository` | No change |

### `CalendarRealtimeSubscription` consumers

| Component | File | How subscribed | Change required |
|-----------|------|----------------|-----------------|
| (not yet wired in any found file) | — | `CalendarRealtimeSubscription.subscribe()` is defined but no call site was found in presentation layer | Confirm no call site before deletion; if found, remove subscription startup |

**Note**: `CalendarRealtimeSubscription` appears to be implemented but not yet wired into the app startup or any screen. Cluster B must confirm this with a grep before deleting the file.

### `CalendarAuditLogger` dependency

`CalendarAuditLogger` is used in `CalendarRepository.saveAvailabilityWindow()` and `saveBlockedTime()`. It writes to local file only (no Supabase). It can be kept as-is after Supabase removal — it has no Supabase dependency.

---

## A4 — Supabase Dependency Map (Cross-referenced by Table)

### Table: `coach_availability`

| Layer | File | Operation | Direction |
|-------|------|-----------|-----------|
| Schema | `20260517150000_create_coach_calendar_schema.sql` | CREATE TABLE, indexes, triggers, RLS | DB |
| Write (queue) | `calendar_sync_queue.dart` `_syncTicket()` | upsert, delete | Flutter → Supabase |
| Enqueue | `calendar_repository.dart` `saveAvailabilityWindow()`, `saveBlockedTime()`, `deleteAvailabilityWindow()`, `deleteBlockedTime()` | Queues for above | Flutter internal |
| Read | **None** | — | — |
| Realtime | **None** | — | — |

### Table: `coach_appointments`

| Layer | File | Operation | Direction |
|-------|------|-----------|-----------|
| Schema | `20260517150000_create_coach_calendar_schema.sql` | CREATE TABLE, indexes, triggers, RLS | DB |
| Write (queue) | `calendar_sync_queue.dart` `_syncTicket()` | upsert, delete | Flutter → Supabase |
| Enqueue | `calendar_repository.dart` `saveAppointmentSlot()`, `deleteAppointmentSlot()` | Queues for above | Flutter internal |
| Read | **None** | — | — |
| Realtime (INSERT) | `calendar_realtime_subscription.dart` `_onInsert()` | SELECT-equiv observer, fires push notification | Supabase → Flutter |
| Realtime (UPDATE) | `calendar_realtime_subscription.dart` `_onUpdate()` | SELECT-equiv observer, fires push notification | Supabase → Flutter |

### Table: `coach_scheduled_checkins`

| Layer | File | Operation | Direction |
|-------|------|-----------|-----------|
| Schema | `20260517150000_create_coach_calendar_schema.sql` | CREATE TABLE, indexes, triggers, RLS | DB |
| Write (queue) | `calendar_sync_queue.dart` `_syncTicket()` | upsert, delete | Flutter → Supabase |
| Enqueue | `calendar_repository.dart` `saveScheduledCheckIn()`, `deleteScheduledCheckIn()` | Queues for above | Flutter internal |
| Read | **None** | — | — |
| Realtime | **None** | — | — |

### Table: `audit_logs`

| Layer | File | Operation | Direction |
|-------|------|-----------|-----------|
| Schema | `20260517150000_create_coach_calendar_schema.sql` | CREATE TABLE IF NOT EXISTS, triggers, RLS | DB |
| Write | Postgres triggers only | INSERT via `log_calendar_audit_event()` trigger function | DB-internal |
| Flutter write | **None** — `CalendarAuditLogger` writes local file only | — | — |
| Read | **None** in current Flutter code | — | — |

---

## Ordered Removal Steps for Cluster B

Based on the actual dependency graph above, the safe removal sequence is:

### B1 — Remove `CalendarSyncQueue` (write side)
- Delete `flutter_app/lib/features/coach/data/calendar_sync_queue.dart`
- Remove all `_syncQueue.enqueue(...)` calls from `CalendarRepository`
- Remove `syncQueue` constructor parameter from `CalendarRepository`
- Remove `initialize()` call from `CalendarTab._initBloc()`
- Remove `syncNow()` / `pendingSyncCount` / `syncStatusStream` from `CalendarRepository` and `AvailabilityService`

**Dependency**: None — can start immediately.

### B2 — Remove sync status UI
- Remove `_buildSyncStatus()` widget from `CalendarTab`
- Remove `import 'calendar_sync_queue.dart' show CalendarSyncQueueStatus` from `calendar_tab.dart`
- Remove `syncStatusStream`, `pendingSyncCount`, `syncNow()` from `CalendarBloc`
- Remove corresponding import from `calendar_bloc.dart`

**Dependency**: B1 must be done first (removes the things these expose).

### B3 — Remove `CalendarRealtimeSubscription` (read/realtime side)
- Confirm no startup wiring exists (grep for `.subscribe()` calls)
- Delete `flutter_app/lib/features/coach/data/calendar_realtime_subscription.dart`
- Replace `CalendarSyncQueue` connectivity detection with `connectivity_plus` package in any remaining connectivity-aware code

**Dependency**: B1. Independent of B2.

### B4 — Replace Supabase connectivity detection
- `CalendarSyncQueue._listenToConnectivity()` used `Supabase.instance.client.realtime` for online/offline detection
- After B1 removes `CalendarSyncQueue`, this is moot — no replacement needed unless a new offline indicator is desired.

**Dependency**: B1 (this is effectively a no-op after B1 deletes CalendarSyncQueue).

### B5 — Down migration (schema removal)
- Run after all runtime references to `coach_availability`, `coach_appointments`, `coach_scheduled_checkins` are removed from Flutter
- Migration must: DROP TABLE `coach_scheduled_checkins`, `coach_appointments`, `coach_availability`, and conditionally remove calendar triggers from `audit_logs`
- Do NOT drop `audit_logs` itself — it may be shared with other subsystems
- Verify no remaining Flutter code references any of the three tables before running

**Dependency**: B1 + B2 + B3 must all be done and verified.

---

## Regression Test Checklist (B5 gate)

Before running the down migration (B5), verify:

- [ ] Coach can create availability windows (Isar only, no Supabase call)
- [ ] Coach can edit and delete availability windows (Isar only)
- [ ] Coach can create and delete blocked time (Isar only)
- [ ] Coach week view renders correctly with local-only data
- [ ] Coach day/month/list views render correctly
- [ ] `CalendarSyncQueueStatus` UI strip is gone (no orphaned StreamBuilder)
- [ ] App does not crash when offline (no Supabase connectivity dependency)
- [ ] `ClientBookingView` loads and renders appointment slots from local Isar
- [ ] `CoachClientWorkspace` book appointment section still works
- [ ] Check-ins section (`_CheckInsSection`) loads from local Isar
- [ ] Push notifications for appointment events are handled gracefully (subscription removed, so no Supabase push — confirm fallback or note as follow-up)
- [ ] No `supabase.from('coach_availability')` calls remain in codebase
- [ ] No `supabase.from('coach_appointments')` calls remain in codebase
- [ ] No `supabase.from('coach_scheduled_checkins')` calls remain in codebase
- [ ] `CalendarRealtimeSubscription` file is deleted and no import remains

---

## Spec Sections Superseded by This Decommission

The following sections of `coach-calendar-spec.md` describe the sync architecture that this decommission removes. They should be marked deprecated once Cluster B is complete:

- §4.3 (Sync queue + Supabase coordination layer) — **superseded**: Isar is now sole source of truth
- §5 (Realtime subscription for appointment push notifications) — **superseded**: realtime channel removed; push notification delivery mechanism needs a follow-up decision (local notification only, or alternative signal)
- Any section referencing `CalendarSyncQueue`, `CalendarSyncTicket`, or `CalendarSyncQueueStatus`

The following sections remain valid:

- §3.1 (Domain models — AvailabilityWindow, AppointmentSlot, ScheduledCheckIn) — local domain unchanged
- §6 (UI / CalendarBloc / CalendarTab) — UI unchanged except sync status strip removal
- §3.2 (Isar local persistence via CalendarLocalStore) — fully retained

## Related

^[source-materials/mirrors/doctrine/supabase-calendar-dependency-map.md]
