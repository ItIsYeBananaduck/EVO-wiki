---
title: "EVOS1-280 — Training Chat Stability Check After Cognition Package Changes"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-05-04
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-280 — Training Chat Stability Check After Cognition Package Changes

## Scope

Issue: **EVOS1-280**  
Date: **2026-05-04**  
Branch in this environment: `work`

Requested checks:
- Pull latest `testing`
- Run EVOtraining locally
- Verify Alice chat opens
- Verify message send/receive
- Verify memory/retrieval paths do not crash
- Verify action adapter changes do not break TalentRegistry/runtime initialization

## Environment outcome

This environment is **blocked by environment** for full acceptance execution.

### Blocker 1 — Cannot pull latest `testing`

Command:

```bash
git fetch origin testing
```

Result:

```text
fatal: 'origin' does not appear to be a git repository
fatal: Could not read from remote repository.
```

Impact: cannot confirm this workspace is aligned to latest `testing`.

### Blocker 2 — Flutter/Dart toolchain unavailable

Commands:

```bash
flutter --version
dart test packages/ai-runtime/test/runtime_sharing_validation_test.dart
```

Result:

```text
/bin/bash: flutter: command not found
/bin/bash: dart: command not found
```

Impact: cannot run EVOtraining locally or execute relevant Dart/Flutter runtime tests in this container.

## Static verification completed

Even with runtime blockers, static checks confirm key wiring surfaces still exist in expected locations:

- `TalentRegistry` runtime usage and tests remain present in action runtime package (`packages/action-runtime/...`).
- iOS runtime wiring references to `TalentRegistry` remain present in the training app runtime (`flutter_app/ios/Runner/...`).
- Shared embeddings package remains present and exports embedding service entrypoints (`packages/domains/cognition_embeddings/...`).

Command used:

```bash
rg -n "TalentRegistry|tool.?to.?action|ActionAdapter|embedding" packages flutter_app apps
```

Representative hits (verified 2026-05-04):

```text
packages/action-runtime/test/talent_definition_test.dart:105:  group('TalentRegistry', () {
packages/action-runtime/test/talent_runner_test.dart:7:      final runner = TalentRunner(talentRegistry: TalentRegistry());
flutter_app/ios/Runner/AliceInferenceManager.swift:99:        let talentRegistry = TalentRegistry()
flutter_app/ios/Runner/ToolCallingFramework.swift:654:    /// Convert tool call to action format (for backwards compatibility)
flutter_app/ios/Runner/AliceInferenceManager.swift:1478:    private func toolInvocationActionAdapter(forTalentId talentId: String) -> [String: String] {
packages/domains/cognition_embeddings/lib/src/embedding_service.dart:22:class EmbeddingService {
packages/domains/cognition_embeddings/lib/src/vector_memory_index.dart:17:/// Stores embedding vectors alongside memory IDs and supports
```

Note: `ActionAdapter` as a standalone class name has no direct hits; the adapter surface is `toolInvocationActionAdapter` in `AliceInferenceManager.swift` (line 1478 above).

## EVOS1-280 acceptance status in this environment

- Training chat opens without crash: **Not validated (blocked by environment)**
- Alice responds to normal message: **Not validated (blocked by environment)**
- No startup/runtime error from missing local embedding classes: **Not runtime-validated; static package presence confirmed**
- No runtime error from shared cognition import wiring: **Not runtime-validated**
- No runtime error from domain-scoped tool-to-action adapter: **Not runtime-validated; static TalentRegistry wiring confirmed**

## Follow-up required

To complete EVOS1-280 acceptance, re-run on a machine with:

1. Valid git remote access to fetch latest `testing`
2. Flutter and Dart SDK installed
3. Ability to launch EVOtraining target and exercise Alice chat interactively

Suggested next commands:

```bash
git fetch origin testing
git switch --track -c testing origin/testing || git switch testing
git pull --ff-only
flutter pub get
flutter run
# then manual chat send/receive and retrieval/action flows
```