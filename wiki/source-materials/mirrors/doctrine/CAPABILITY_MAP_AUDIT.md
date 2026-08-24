---
title: CAPABILITY_MAP_AUDIT
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CAPABILITY_MAP_AUDIT.md"]
updated: 2026-07-24
---

# Capability Map Audit - Complete Function List

## ✅ All Functions Mapped

### Core Agentic Actions (8)

1. ✅ `none` - No action
2. ✅ `navigate` - Screen navigation
3. ✅ `update_workout_plan` - Create/update workout plans
4. ✅ `update_nutrition_targets` - Set nutrition goals
5. ✅ `adjust_live_rest` - Rest time adjustments
6. ✅ `enforce_deload_stop` - Safety deloads
7. ✅ `schedule_mesocycle_update` - Schedule updates
8. ✅ `update_profile` - Profile changes

### AI Plan Operations (7) - Pro-only

9. ✅ `plan.create` - Create new workout plan (AI)
10. ✅ `plan.generate` - Generate workout plan (AI)
11. ✅ `plan.edit_major` - Major plan edits (AI)
12. ✅ `plan.modify` - Modify existing plan (AI)
13. ✅ `plan.swapExercise` - Swap exercise in plan (AI)
14. ✅ `plan.autoprogress` - Auto-progress plan (AI)
15. ✅ `plan.deload` - Deload plan (AI)

### Admin-Only Actions (2)

16. ✅ `admin.override` - Admin override
17. ✅ `system.config` - System configuration

### Automatic Actions (9)

1. ✅ `strainBasedRestAdjustment` - Strain-based rest (PRIMARY)
2. ✅ `weekOverWeekProgression` - Week-over-week progressions
3. ✅ `mesocycleTransition` - Mesocycle transitions
4. ✅ `exerciseReplacement` - Exercise replacements
5. ✅ `volumeAdjustment` - Volume adjustments
6. ✅ `systemDeload` - System-triggered deloads
7. ✅ `alicePerceivedIntensity` - Alice's intensity score
8. ✅ `musicPerformanceCorrelation` - Music-performance analysis
9. ✅ `performancePlaylistCreation` - Performance playlist creation

## Sources Checked

### ✅ actions_schema.json

- All 8 core actions mapped

### ✅ actions_schema.dart

- All plan.\* operations mapped (7 actions)
- Admin actions mapped (2 actions)

### ✅ action_catalog.dart

- Verified all action types match

### ✅ AUTO_AGENTIC_ACTIONS_AUDIT.md

- All automatic actions mapped

### ✅ agentOrchestrator.ts

- Rest decisions mapped
- Plan decisions mapped
- Volume adjustments mapped

## Total Count

- **Agentic Actions**: 17 (8 core + 7 plan ops + 2 admin)
- **Automatic Actions**: 9
- **Total Functions**: 26

## Verification

All functions from:

- ✅ `actions_schema.json`
- ✅ `actions_schema.dart`
- ✅ `action_catalog.dart`
- ✅ `AUTO_AGENTIC_ACTIONS_AUDIT.md`
- ✅ `agentOrchestrator.ts`

Have been mapped in `alice_capability_map.json`

## Missing Functions Check

If you find any additional functions that should be mapped, add them to this audit and update the capability map.

## Related
