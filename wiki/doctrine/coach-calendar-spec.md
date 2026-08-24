---
title: coach-calendar-spec
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/coach-calendar-spec.md
updated: 2026-07-24
---

# Coach Calendar — Implementation Spec

## Runtime Ownership

Coach Calendar belongs entirely to EVOtraining (`flutter_app/`).

It is not an EVOconnect runtime feature.
EVOconnect may surface coach availability later through Pane Packs and marketplace discovery, but the source-of-truth runtime remains EVOtraining.

## Ownership Statement

This document defines the implementation contract for the **Calendar** tab in the Coach Application. It is a first-class navigation destination distinct from Plan Builder timeline views.

## Local-First Architecture Boundary

Coach Calendar follows EVOsystem local-first doctrine.

Supabase is used for:
- authentication
- encrypted synchronization
- cross-device continuity
- coach/client relationship coordination
- optional marketplace visibility
- notification coordination

Supabase is NOT the primary runtime.

Primary ownership of calendar state belongs to the coach device.
The local device remains authoritative during active sessions.

The calendar system must:
- function offline
- queue sync operations when connectivity is unavailable
- reconcile changes after reconnect
- allow coaches to operate without continuous cloud connectivity

Client-visible availability may be synchronized to Supabase for booking discovery, but private operational calendar state should remain local unless explicitly required for synchronization.

Calendar architecture must avoid turning Supabase into a permanent execution dependency.

---

## 1. What the Calendar Tab Owns

### 1.1 Core Responsibilities
- **Coach Availability Windows**: Recurring and one-off availability slots when the coach can accept appointments
- **Appointment Slots**: Booked and pending client appointments (check-ins, consultations, reviews)
- **Scheduled Check-ins**: Automated or manually scheduled client touchpoints tied to calendar time slots
- **Blocked Time**: Personal time, travel, non-client work (visible to coach only)

### 1.2 Entity Types
| Entity | Visibility | Mutable By |
|--------|------------|------------|
| AvailabilityWindow | Coach only | Coach |
| AppointmentSlot | Coach + Client | Coach (creates), Client (books within open slots) |
| ScheduledCheckIn | Coach + Client | Coach, System (automated) |
| BlockedTime | Coach only | Coach |

---
## Alice Boundaries

Alice may:
- schedule automated check-ins
- suggest availability optimizations
- propose appointment slots
- remind coaches of upcoming sessions

Alice may not:
- confirm appointments autonomously unless coach delegation permits it
- modify blocked time without approval
- expose private coach availability to unrelated clients

## 2. Explicitly Out of Scope

The Calendar tab does **NOT** own:

- **Plan Timelines**: Periodization phases, deload weeks, mesocycle boundaries — these belong to **Plan Builder**
- **Workout Scheduling**: Which workout happens on which day — this is a **Plan → Client** concern, not calendar scheduling
- **Exercise History**: Past performance logs — this is **Workout History**
- **Program Publishing**: When a plan becomes available to clients — this is **Plan Builder → Publish**

> Boundary rule: If it relates to "what the client does" (exercises, workouts, nutrition), it belongs elsewhere. Calendar owns "when the coach and client meet."

---

## 3. Calendar Data Model

### 3.1 Sync Schema (Supabase)

These tables represent synchronization and coordination state — not the authoritative in-memory runtime.

Coach devices maintain local-first calendar state and synchronize selectively to Supabase for:
- cross-device continuity
- client booking visibility
- appointment coordination
- notification delivery
- marketplace discovery

```sql
-- Core calendar entities
create table coach_availability (
    id uuid primary key default gen_random_uuid(),
    coach_id uuid references auth.users(id) not null,

    -- Time range
    starts_at timestamptz not null,
    ends_at timestamptz not null,

    -- Recurrence (optional)
    recurrence_rule text, -- RFC 5545 RRULE string, null for one-off
    recurrence_id uuid references coach_availability(id), -- link to parent for recurring series

    -- Metadata
    slot_type text not null check (slot_type in ('available', 'blocked')),
    location text, -- 'video', 'in_person', 'phone', or address
    notes text,

    created_at timestamptz default now(),
    updated_at timestamptz default now()
);

create table coach_appointments (
    id uuid primary key default gen_random_uuid(),
    coach_id uuid references auth.users(id) not null,
    client_id uuid references profiles(id),
    availability_slot_id uuid references coach_availability(id),

    -- Time (denormalized from availability for query efficiency)
    starts_at timestamptz not null,
    ends_at timestamptz not null,
    timezone text not null default 'America/New_York',

    -- Type and status
    appointment_type text not null check (appointment_type in (
        'initial_consult', 'check_in', 'plan_review', 'form_review', 'nutrition_review', 'other'
    )),
    status text not null default 'pending' check (status in ('pending', 'confirmed', 'completed', 'cancelled', 'no_show')),

    -- Booking
    requested_by uuid references auth.users(id), -- who initiated
    requested_at timestamptz,
    confirmed_at timestamptz,
    cancelled_at timestamptz,
    cancellation_reason text,

    -- Session data
    meeting_url text, -- video link
    location_address text, -- for in-person
    pre_session_notes text, -- coach prep
    post_session_notes text, -- coach notes after
    client_prep_notes text, -- sent to client before

    created_at timestamptz default now(),
    updated_at timestamptz default now()
);

create table coach_scheduled_checkins (
    id uuid primary key default gen_random_uuid(),
    coach_id uuid references auth.users(id) not null,
    client_id uuid references profiles(id) not null,
    plan_assignment_id uuid references client_plan_assignments(id),

    -- Scheduling
    scheduled_at timestamptz not null,
    timezone text not null,
    checkin_type text not null check (checkin_type in ('weekly', 'biweekly', 'monthly', 'milestone', 'custom')),

    -- Status
    status text not null default 'scheduled' check (status in ('scheduled', 'completed', 'skipped', 'rescheduled')),
    completed_at timestamptz,

    -- Content
    discussion_points text[], -- agenda items
    client_metrics_snapshot jsonb, -- weight, body comp at time of checkin
    notes text,

    created_at timestamptz default now(),
    updated_at timestamptz default now()
);

## Audit Events

The following actions must emit audit events:
- availability created
- availability modified
- appointment requested
- appointment confirmed
- appointment cancelled
- check-in completed
- blocked time created

```

### 3.2 Row Level Security (RLS)

```sql
-- Coaches see their own calendar data
alter table coach_availability enable row level security;
create policy coach_sees_own_availability on coach_availability
    for all using (coach_id = auth.uid());

-- Coaches see their own appointments; clients see their own appointments
alter table coach_appointments enable row level security;
create policy coach_sees_own_appointments on coach_appointments
    for all using (coach_id = auth.uid());
create policy client_sees_own_appointments on coach_appointments
    for select using (client_id in (
        select id from profiles where user_id = auth.uid()
    ));

-- Same pattern for check-ins
alter table coach_scheduled_checkins enable row level security;
create policy coach_sees_own_checkins on coach_scheduled_checkins
    for all using (coach_id = auth.uid());
create policy client_sees_own_checkins on coach_scheduled_checkins
    for select using (client_id in (
        select id from profiles where user_id = auth.uid()
    ));
```

---

## 4. Client Workspace Integration

### 4.1 What Clients See
- Their own appointments (not coach's full calendar)
- Available slots for booking (coach-published availability only)
- Their scheduled check-ins (read-only)

### 4.2 Booking Flow
1. Client views available slots (filtered to their timezone)
2. Client selects slot → triggers booking request
3. Coach receives notification (in-app + optional push)
4. Coach confirms → slot becomes appointment
5. System generates calendar invite (.ics) for both parties

### 4.3 Data Flow Diagram
```
┌─────────────────┐
│ Coach Device    │
│ (authoritative) │
└────────┬────────┘
         │
         │ local-first runtime
         ▼
┌─────────────────┐
│ Local Calendar  │
│ Store / Hive    │
└────────┬────────┘
         │
         │ selective sync
         ▼
┌─────────────────┐     ┌─────────────────┐
│   Supabase      │◄───►│   Client App    │
│ sync + booking  │     │ limited surface │
└─────────────────┘     └─────────────────┘
```

---

## 5. EVOconnect Calendar Surface (Future)

### 5.1 Relationship
- **Coach Calendar** = single coach view, desktop-optimized
- **EVOconnect Calendar** = multi-coach marketplace, client-optimized

### 5.2 Shared Schema Strategy
- `coach_availability` and `coach_appointments` are designed to support multi-coach queries
- EVOconnect will add `marketplace_listings` and `coach_profiles_public` tables
- Same appointment booking flow, but initiated from marketplace context
- EVOconnect never becomes the authoritative calendar runtime; it surfaces synchronized availability only

### 5.3 Migration Path
- No schema changes required when adding EVOconnect
- Add `marketplace_visible` boolean to `coach_availability` (default false)
- Add `booking_source` enum ('direct', 'marketplace') to `coach_appointments`

---

## 6. UI/UX Structure

### 6.1 Calendar Views (Coach Desktop)
- **Day view**: Detailed slot management, drag-to-resize
- **Week view**: Primary working view, see pattern of availability
- **Month view**: High-level overview, appointment density
- **List view**: Agenda-style, upcoming appointments

### 6.2 Key Interactions
| Action | Gesture | Notes |
|--------|---------|-------|
| Create availability | Click empty slot | Drag to set duration |
| Edit slot | Click existing | Modal with full controls |
| Convert to appointment | Click + "Book" | Opens client selector |
| Block time | Right-click → Block | Same as availability, type=blocked |
| Reschedule | Drag to new time | Maintains client association |

### 6.3 Visual States
- Available: Green outline, clickable
- Booked (confirmed): Solid blue, client avatar
- Booked (pending): Striped yellow, "pending" badge
- Blocked: Gray background, no interactions
- Past: Reduced opacity, completed checkmark if done

---

## 7. API Surface

### 7.1 Sync API Surface (Supabase functions)

These endpoints represent synchronization and booking coordination APIs. They are not the local runtime API. Local calendar operations should execute against the local store first, then enqueue sync work when needed.

```typescript
// Availability
GET  /coach/availability?start=2026-06-01&end=2026-06-30
POST /coach/availability
PATCH /coach/availability/:id
DELETE /coach/availability/:id

// Appointments
GET  /coach/appointments?start=...&end=...
GET  /coach/appointments/:id
POST /coach/appointments
PATCH /coach/appointments/:id/status
DELETE /coach/appointments/:id

// Check-ins
GET  /coach/checkins?client_id=...
POST /coach/checkins
PATCH /coach/checkins/:id/complete

// Client-facing (limited)
GET  /client/availability?coach_id=...&start=...&end=...
POST /client/appointments/request  // creates pending appointment
GET  /client/appointments
```

### 7.2 Realtime Subscriptions
- Coach subscribes to own `coach_appointments` for live updates
- Client subscribes to own appointments for status changes
- Optional: Subscribe to availability changes (if coach modifies open slots)

Realtime subscriptions are synchronization helpers only.
The calendar system must continue functioning locally if subscriptions fail or connectivity is interrupted.

---

## 8. Implementation Phases

### Phase 0: Local Runtime & Sync Layer
- Define local calendar persistence layer (Hive/local DB)
- Define sync queue architecture
- Define conflict reconciliation strategy
- Define offline-first behavior
- Define sync boundaries for private vs shared calendar state

### Phase 1: Schema + RLS (Week 1)
- Create tables with full RLS policies
- Seed with test data
- Verify security with policy tests

### Phase 2: Core CRUD (Week 2)
- Availability management (create, edit, delete)
- Basic calendar views (week view primary)
- Simple appointment creation

### Phase 3: Client Integration (Week 3)
- Client slot booking flow
- Appointment notifications
- Calendar invite generation

### Phase 4: Polish (Week 4)
- Recurring availability
- Drag-and-drop rescheduling
- Check-in scheduling
- Month/day/list views

---

## 9. Decisions

1. **Video integration**: No native in-app video integration for this phase. Coach Calendar may store an optional external meeting URL, but video hosting is not owned by EVOtraining.
2. **Notifications**: Push notifications only. SMS and email reminders are out of scope unless added by a later notification spec.
3. **Calendar sync**: Yes. Coach Calendar should support external calendar sync/export, including Google/Outlook-style calendar interoperability, without making external calendars the source of truth.
4. **Timezones**: Yes. Store canonical event times in UTC and convert to the user's local timezone on read/display. Preserve timezone metadata for appointments and recurring availability.

---

## 10. Acceptance Criteria

- [ ] Schema migrations applied with zero errors
- [ ] RLS policies block unauthorized cross-coach access
- [ ] Coach can create, view, edit, delete availability windows
- [ ] Coach can convert availability to appointment with client
- [ ] Client can view available slots and request bookings
- [ ] Realtime updates appear within 2 seconds
- [ ] Calendar functions without active internet connectivity
- [ ] Local changes queue safely during offline operation
- [ ] Sync reconciliation resolves conflicts deterministically
- [ ] Supabase outage does not prevent coach calendar operation
- [ ] All calendar actions emit audit logs

---

**Status**: Draft — approved by @phillip
**Last Updated**: 2026-05-16
**Related**: [Coach Application Philosophy](link-to-philosophy-doc)

## Related

^[source-materials/mirrors/doctrine/coach-calendar-spec.md]
