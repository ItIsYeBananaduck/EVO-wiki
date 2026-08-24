---
title: coach-planning-spec
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/coach-planning-spec.md"]
updated: 2026-07-24
---

# Coach Planning Spec

## Purpose

This planning spec defines architectural and execution boundaries for the Coach module during implementation planning and issue generation.

Coach is an EVOtraining subdomain/module. In this repository, EVOtraining lives in `flutter_app/`.

Coach is not a separate app, separate runtime, separate repository area, separate AI system, or separate orchestration layer. Coach must be implemented inside the existing EVOtraining architecture.

## Repository Mapping

Coach implementation planning must assume shared Flutter runtime, shared Supabase foundations, shared training models, shared workout/session systems, shared talent orchestration, shared cognition systems, shared adapters, shared journals/logging systems, and shared AI orchestration.

Do not generate plans that duplicate existing EVOtraining infrastructure unless explicitly required by doctrine.

## Reuse Requirements

Planning agents must reuse existing EVOtraining systems wherever possible, including workout runtime, session execution, training logs, journals, adaptation systems, talent orchestration, trainer adapters, AI runtime surfaces, auth/subscription foundations, shared UI systems, Hive/runtime orchestration, and cognition infrastructure.

## Coach-Specific Scope

Coach-specific implementation should only be introduced where required by coach/client relationships, permissions boundaries, coach-facing UI, client management, plan assignment, plan review, progress review, trainer methodology influence, trainer adapter influence boundaries, coach analytics, marketplace integration, or Trainer Link workflows.

## Trainer Link Boundary

Trainer Link is a coach-client inquiry and subscription handoff workflow.

Trainer Link should begin as a chat entry point for a trainer. A prospective or existing client can message the trainer, the trainer can ask follow-up questions, and the trainer can decide what subscription or coaching offer fits that client.

The trainer subscription link is sent through that trainer-client chat after the trainer has enough context.

Trainer Link is not just a static payment link and should not be planned as a checkout-only shortcut.

## Coach Onboarding Boundary

A user must sign up as a trainer/coach before using Coach-facing operational surfaces.

Coach onboarding must include certification submission. Certification is part of becoming a trainer in the EVOtraining ecosystem, not merely an optional marketplace badge.

Implementation planning should account for trainer identity, certification capture, certification review status, and access to Coach surfaces only after the appropriate onboarding state exists.

## Coach Calendar Boundary

The Coach Calendar may include scheduling, appointments, availability, workout timing, and coach/client coordination.

The Coach Calendar must not own plan timelines.

Plan timelines belong to Plan Builder, mesocycle structure, week structure, and training plan history.

If calendar work is planned, it must avoid duplicating Plan Builder responsibility.

## Coach Pane Pack Boundary

Coach may eventually expose a Pane Pack surface for EVOconnect, but this remains cross-domain architecture.

Implementation planning should not assume full Connect hosting behavior until a Coach Pane Pack Contract exists.

Coach Pane Pack work should be treated as doctrine-first unless a canonical contract is supplied.

## Planning Constraints

Planning agents must not split Coach into a separate application, create duplicate training runtimes, duplicate EVOtraining talents, duplicate AI orchestration, duplicate training session systems, duplicate logging systems, or create parallel adapter systems.

Coach extends EVOtraining. Coach does not replace or fork EVOtraining.

## Deferred Work Handling

When evo-plan defers Coach work, every deferred item must receive a next-state.

Use these categories:

- Doctrine-first defer: missing scope, ownership, data model, permission model, or architectural contract.
- Follow-up implementation defer: doctrine is clear, but implementation is intentionally outside the current cluster.
- Not-now scope: intentionally excluded because it would bloat the current module or violate the planning spec.

Deferred work should identify the missing doctrine or future implementation boundary rather than becoming a vague parking lot.

## Linear Planning Rules

Default project: EVOtraining.

Shared infrastructure blockers: EVOsystem.

Planning hierarchy should prefer runtime reuse audits first, then data model extension, permission systems, coach UI surfaces, orchestration extensions, and analytics/review surfaces.

## evo-plan Behavior

When evo-plan is invoked with this spec, it must load this planning spec first, perform LLM wiki discovery across canonical notes, identify reusable systems first, identify missing doctrine second, identify implementation clusters third, avoid generating duplicate infrastructure, and prioritize integration over replacement.

The primary question is: How does Coach integrate into existing EVOtraining architecture?

## Related
