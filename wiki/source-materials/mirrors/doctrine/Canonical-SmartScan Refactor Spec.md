---
title: Canonical-SmartScan Refactor Spec
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Canonical-SmartScan Refactor Spec.md"]
updated: 2026-07-24
---

# Canonical-SmartScan Refactor Spec
Planning Spec: Canonical SmartScan Refactor

Purpose:
Refactor the existing SmartScan implementations into one canonical SmartScan system that supports nutrition/supplement scanning, body composition scanning, and workout importing without duplicating camera/OCR/review/import logic.

Current Problem:
There are currently separate SmartScan flows:
1. Nutrition/Supplement SmartScan
2. Body Composition SmartScan

Workout importing now needs the same scan/OCR/parsing/review/import flow.

We do NOT want to create a third duplicated SmartScan path. We also do NOT want the old implementations left behind in active code, because duplicate scan paths will drift, confuse agents, and poison future refactors.

Goal:
Create one shared SmartScan pipeline with profile-based behavior.

Canonical Architecture:
SmartScan Core:
- camera/session setup
- capture mode handling
- image/frame normalization
- OCR invocation
- parsing pipeline orchestration
- confidence scoring
- validation orchestration
- review-required decisioning
- result envelope generation
- import handoff routing

SmartScan Profiles:
- nutrition_supplement
- body_composition
- workout_import

Each profile owns:
- expected fields/schema
- domain-specific parsing/mapping
- validation rules
- confidence thresholds
- review UI configuration/copy
- destination import target
- profile-specific metadata

Required Capture Modes:
- silent_frame_capture for repeated physical scanning
- manual_photo_capture for explicit user photos
- document_scan if already available or easy to support
- barcode_scan as future-compatible profile/capture support

Important UX Requirement:
Continuous physical scanning must not repeatedly trigger the camera shutter sound.
If the current implementation uses normal photo capture APIs that trigger shutter sounds, refactor continuous scan mode to use frame capture from the camera stream or another silent capture path.

Migration Strategy:
1. Inventory existing nutrition/supplement SmartScan code.
2. Inventory existing body composition SmartScan code.
3. Identify shared logic vs profile-specific logic.
4. Create canonical SmartScan module/folder.
5. Move shared logic into SmartScan Core.
6. Convert nutrition/supplement scanning into a SmartScanProfile.
7. Convert body composition scanning into a SmartScanProfile.
8. Add workout_import as a new SmartScanProfile.
9. Update all callers to use the canonical SmartScan entrypoint.
10. Remove or deprecate old duplicate SmartScan files after migration.
11. Add safeguards/tests so future agents do not recreate duplicate scan paths.

Non-Goals:
- Do not redesign the entire camera system unless required.
- Do not change destination models unless needed for compatibility.
- Do not leave legacy scan paths active.
- Do not create SmartScanNutrition, SmartScanBodyComp, and SmartScanWorkout as separate engines.
- Do not silently bypass review-required behavior.

Acceptance Criteria:
1. There is one canonical SmartScan entrypoint.
2. Nutrition/supplement scan uses the canonical pipeline.
3. Body composition scan uses the canonical pipeline.
4. Workout import uses the canonical pipeline.
5. Shared camera/OCR/review/import orchestration is not duplicated.
6. Profile-specific logic is isolated in profiles/adapters.
7. Old duplicate SmartScan implementations are removed or clearly marked deprecated and unused.
8. Continuous scanning does not repeatedly trigger shutter sounds.
9. Tests cover:
   - profile registration
   - profile selection
   - OCR result mapping
   - confidence scoring
   - validation failures
   - review-required decisions
   - import handoff
10. A code search for old SmartScan entrypoints confirms no active callers remain.

Agent Instructions:
Before coding, produce an implementation plan that lists:
- all current SmartScan-related files
- which files are shared logic
- which files are profile-specific
- proposed new canonical folder/module structure
- migration order
- deletion/deprecation plan
- tests to add/update

Do not begin implementation until the plan proves that duplicate scan paths will be removed or fully disconnected.

## Related

^[{src_rel}]
