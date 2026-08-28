---
title: "EVOS1-387 — MLX Download Gating Runtime"
type: spec
tags: ['lsctech', 'spec', 'source-material', 'canonical', 'evo']
updated: 2026-08-19
---

# EVOS1-387 — MLX Download Gating Runtime

EVOS1-387 implements the MLX download-gating recommendation from the EVOS1-386 analysis as an opt-in Apple-only runtime path, not a default engine replacement.

## Implemented Decisions

- MLX eligibility is centralized in `AliceMlxEligibility`, with typed eligible/ineligible results and ordered checks for platform, OS, Metal family, unified memory, disk, denylist, thermal/battery pressure, and app mode.
- The R2 MLX model contract is represented by `MlxModelManifest` and `scripts/mlx/manifest/schema.json`, with mandatory checksum, runtime type, version, rollback, platform, app-version, memory, disk, and denylist fields.
- Apple device facts are gathered by `MlxEligibilityProbe` in `EffectiveConfig.swift`; Lite Mode exposes `isMlxEligibleMode` so callers do not re-derive mode state.
- `AliceAssetDownloadManager` now fetches and validates the MLX manifest, verifies checksums strictly, records verified MLX state, exposes typed download-unavailability reasons, and deletes corrupt checksum failures before re-exposing download.
- `AliceInferenceManager.Engine` now supports `mlx`, selected only when the user opted in, the asset is verified, and the launch-time eligibility gate still passes; fallback remains GGUF/llama.cpp or none.
- `MLXEngine` mirrors the existing Metal/Impeller load guard before model load on iOS 26.x.
- MLX personalization uses `MlxRetrievalBriefService`, backed by journal entries indexed through `ConversationMemoryManager`, with retrieval failures degrading to an empty brief rather than blocking inference.
- The fallback matrix is covered by `alice_mlx_fallback_test.dart`, including sub-threshold devices, denylist changes, manifest fetch failure, app-version gating, checksum deletion, disk loss, launch thermal pressure, and retrieval unavailability.

## Validation Evidence

- `flutter analyze lib/features/alice/domain/mlx/`
- `flutter test flutter_app/lib/features/alice/domain/mlx/mlx_manifest_test.dart`
- `flutter test lib/features/alice/domain/mlx/mlx_manifest_test.dart`
- `swiftc -typecheck flutter_app/ios/Runner/EffectiveConfig.swift`
- Worker-reported MLX-related test passes for EVOS1-390 and EVOS1-392

## Remaining Boundary

Physical-device benchmark evidence from EVOS1-379 remains pending. This implementation gates and falls back safely, but does not promote MLX to a default runtime.