---
title: trainer-calendar-supabase-decommission
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/trainer-calendar-supabase-decommission.md"]
updated: 2026-07-24
---

# Planning Spec — Trainer Calendar Supabase Decommission

## Purpose

Refactor the trainer calendar so Supabase is no longer the source of truth for calendar mechanics unless cloud coordination is explicitly required.

The app should preserve trainer scheduling, client visibility, local-first behavior, and workout result handoff while removing unnecessary Supabase calendar state.

## Core Problem

The calendar system currently has Supabase-backed pieces that may no longer match the intended product model.

The client app was never intended to own a full calendar. The trainer schedules sessions, and the client should see assigned sessions, workout expectations, and results.

If Supabase calendar state is not truly needed, keeping it creates unnecessary complexity, sync bugs, and future maintenance risk.

## Canonical Direction

Calendar mechanics should be local-first.

Supabase should only store relationship-level training data when needed, such as trainer/client assignments, subscriptions, shared plans, assigned sessions, and workout results.

Supabase should not own calendar CRUD unless a cloud-shared calendar feature becomes explicitly required later.

## Required Audit

Before removing anything, audit:

- Supabase tables used by calendar logic
- migrations
- services
- repositories
- RPCs
- edge functions
- Flutter calendar providers
- Swift/iOS calendar bridges
- Android/Kotlin calendar assumptions
- ISR/local sync behavior
- trainer scheduling flow
- client session visibility flow
- workout result handoff

## Refactor Rules

Do not delete first.

First prove:

- what currently writes calendar state
- what currently reads calendar state
- what the UI depends on
- what local storage already supports
- what the trainer actually needs
- what the client actually sees

Then replace Supabase calendar reads/writes with the local/session model.

Only after runtime references are removed should Supabase calendar schema and services be deleted.

## Desired Model

Trainer side:

- schedule sessions
- assign workouts
- view upcoming client work
- view completed results

Client side:

- see assigned sessions
- see workout expectations
- complete workouts
- view results/history

Shared state:

- trainer/client relationship
- assigned plans
- assigned sessions if cloud sharing is required
- workout completion/result records

Not shared state:

- full calendar CRUD
- generic calendar sync
- client-owned calendar management
- Supabase-owned calendar mechanics

## Deliverables

1. Calendar ownership audit.
2. Supabase dependency map.
3. Local/session replacement model.
4. Refactor implementation plan.
5. Dead-code removal checklist.
6. Regression test checklist.

## Regression Coverage

Verify:

- trainer can schedule sessions
- client can see assigned sessions
- offline/local behavior still works
- workout completion still creates results
- trainer can view client results
- no Supabase calendar calls remain in runtime paths
- app does not crash when old calendar cloud data is absent
- migrations do not leave orphaned runtime dependencies

## Success Criteria

The calendar works without Supabase calendar state.

Supabase remains available only for relationship-level training data where cloud coordination is actually needed.

No zombie calendar code remains.

## Related
