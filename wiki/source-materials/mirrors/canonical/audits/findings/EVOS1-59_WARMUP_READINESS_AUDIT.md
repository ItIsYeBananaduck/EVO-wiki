---
title: "EVOS1-59 Audit: Alice warmup + readiness lifecycle"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-05-12
---

> **Status: Historical Reference**
> Audit of Alice warmup + readiness lifecycle bug (racy first-message behavior). Investigation complete. Historical bug investigation record.

# EVOS1-59 Audit: Alice warmup + readiness lifecycle

## Scope audited

- Chat send flow in Flutter (`_sendMessage`, `_sendMessageWithRaw`) in:
  - `flutter_app/lib/features/alice/presentation/alice_chat_screen.dart`
- Native warmup dispatch and readiness gate in:
  - `flutter_app/ios/Runner/AliceInferenceManager.swift`
  - `flutter_app/ios/Runner/AppDelegate.swift`
- Native generation queue/guard behavior in:
  - `flutter_app/ios/Runner/LlamaEngine.swift`
- Dart-to-native generation bridge in:
  - `flutter_app/lib/features/alice/domain/alice_brain_service.dart`

---

## Executive summary (single source of truth)

The first-message blocking behavior is caused by a **lifecycle mismatch**:

1. Native marks inference as "ready" immediately after model load (`initState = ready`) and triggers warmup (`"hi"`) asynchronously.
2. Warmup is **fire-and-forget** (response never bridged to Flutter chat state).
3. User requests arriving before readiness is synchronized are handled by two disjoint paths:
   - Flutter `alice_chat_screen` path: one-slot pending message buffer (`_pendingWarmupMessage`) with replay only on specific readiness checks.
   - Native `generate(...)` path: if not ready, returns lightweight placeholder text (`"Give me a second, I’m getting ready."`) and does **not queue/replay** original request.
4. Because these paths are not unified, the first real message can be deferred, replaced, or consumed by a non-ready fallback without deterministic replay.

In short: there is no single authoritative request state machine spanning Flutter + native warmup + native readiness fallback.

---

## Detailed lifecycle findings

## 1) Where warmup is triggered and how it runs

Warmup is triggered in native iOS manager after model load success:

- `AliceInferenceManager.doInitialize()` → `runWarmupInference()`
- `AliceInferenceManager.initializeAsync(...)` success path → `runWarmupInference()`

Warmup payload is synthetic:

- `sessionId: "alice_warmup"`
- `domain: "warmup"`
- `userMessage: "hi"`

Warmup runs on a background queue and calls `LlamaEngine.shared.generate(...)` directly.

**Critical observation:** warmup completion callback only logs. It does not route the response to Flutter chat UI, chat history, or pending-message replay logic.

### Answer: Does warmup consume first request / run before / parallel?

- Warmup starts as soon as native init reaches ready and runs asynchronously.
- Real user requests can be sent during or after this interval depending on Flutter readiness state.
- Effective behavior is **parallel/racy**, not strictly serialized behind warmup.

---

## 2) Flutter chat gating and pending message behavior (`alice_chat_screen`)

In `alice_chat_screen.dart`:

- `_sendMessage()` blocks on:
  - `!_historyInitialized`
  - `_modelLoading`
- If input exists but `_inferenceReady == false`, it calls `_queueMessageDuringWarmup(raw)`.

Queue behavior:

- `_pendingWarmupMessage` is a **single slot**.
- Additional sends while pending are ignored.
- The queued message is rendered with `isWarmupPending: true`.

Replay behavior:

- `_dispatchPendingWarmupMessage()` sends queued content only when all are true:
  - `_inferenceReady == true`
  - `_pendingWarmupMessage != null`
  - `_sending == false`
- Replay is triggered from `_checkModelFiles()` when diagnostic status reports loaded.

**Consequence:** replay is tied to periodic readiness polling/status transitions, not to a canonical warmup-complete event.

---

## 3) Native non-ready behavior drops replay responsibility

In `AliceInferenceManager.generate(...)`:

- Guard checks `initState == .ready && isReady()`.
- If not ready, it calls `startBackgroundInitialization()` and returns a lightweight response:
  - `"Give me a second, I’m getting ready."`
- No pending queue for real user request is stored in `AliceInferenceManager`.
- No auto replay is attempted after initialization/warmup completes.

This is the main reason a user can need to resend: the original semantic intent is not retained by native when non-ready guard trips.

---

## 4) Native generation queue exists but is scoped incorrectly for startup readiness

`LlamaEngine` has `pendingRequest` queueing only when a generation is already active (`state == .generating`), and it may cancel previous request then run pending.

This queue does **not** capture requests rejected earlier by `AliceInferenceManager.generate(...)` non-ready guard.

Therefore:

- Queue helps with overlap/concurrency once generation started.
- Queue does not solve startup readiness/warmup barrier handoff.

---

## Direct answers to requested questions

## Q1. Is first user message ignored, queued, overwritten by warmup, or blocked?

- In `alice_chat_screen` path, first message is usually **queued** (single-slot) when `_inferenceReady == false`.
- In native non-ready path, first message is effectively **blocked and replaced by placeholder response** (no replay memory).
- Not literally overwritten by warmup content, but warmup can occupy early lifecycle while real request lacks canonical replay ownership.

## Q2. Does warmup consume first request, run before, or run in parallel?

- Warmup runs **in parallel/asynchronously** once native is marked ready.
- It is not guaranteed to run strictly before all user messages.
- It does not consume chat request directly, but contributes to race conditions in startup lifecycle.

## Q3. Why warmup-generated response not shown in chat?

Because warmup uses direct native `LlamaEngine.shared.generate(...)` with logging-only completion; there is no method-channel propagation to Flutter chat messages/history.

## Q4. Why real user message not automatically replayed after warmup?

Because replay is fragmented:

- Flutter chat has one local pending slot replayed only on `_checkModelFiles()` readiness transitions.
- Native non-ready guard returns lightweight text and does not store/replay original request.
- No single end-to-end queue/state machine exists across warmup + readiness + request dispatch.

---

## Exact code locations responsible for block/loss risk

1. **Synthetic warmup dispatch (fire-and-forget)**
   - `flutter_app/ios/Runner/AliceInferenceManager.swift`
   - `runWarmupInference()` (uses `userMessage: "hi"`, no UI commit path)

2. **Native non-ready guard returns placeholder instead of queuing user request**
   - `flutter_app/ios/Runner/AliceInferenceManager.swift`
   - `generate(...)` readiness guard (`state == .ready && isReady()`) + `lightweightResponse(...)`

3. **Flutter pending queue is single-slot and local only**
   - `flutter_app/lib/features/alice/presentation/alice_chat_screen.dart`
   - `_queueMessageDuringWarmup(...)`, `_pendingWarmupMessage`, `_dispatchPendingWarmupMessage(...)`

4. **Replay trigger is polling/state dependent, not warmup-complete event driven**
   - `flutter_app/lib/features/alice/presentation/alice_chat_screen.dart`
   - `_checkModelFiles()` + `_setInferenceReadiness(...)` + readiness poll timer

5. **Native queue exists only for in-flight generation overlap**
   - `flutter_app/ios/Runner/LlamaEngine.swift`
   - `pendingRequest` handling in generation state machine cleanup

---

## Lifecycle diagram (current)

```text
App start
  -> Native initializeAsync()
    -> initState=loading
    -> loadModel()
    -> initState=ready
    -> runWarmupInference("hi") [background, no UI commit]

User sends first message
  A) Alice chat screen and _inferenceReady=false
     -> queue local _pendingWarmupMessage (single slot)
     -> later _checkModelFiles() sees loaded
     -> _dispatchPendingWarmupMessage() replays

  B) Request reaches native generate while init not ready
     -> lightweightResponse("Give me a second...")
     -> no native pending queue for semantic replay
     -> user must resend

Warmup completion
  -> logs only
  -> no direct trigger to flush/replay unified pending user requests
```

---

## Recommended fix strategy (do not implement yet)

1. **Introduce a single source-of-truth request state machine**
   - One canonical queue for user requests before inference readiness.
   - Ownership should be clear (prefer native, with explicit ack back to Flutter).

2. **Separate readiness into two explicit states**
   - `modelLoaded`
   - `warmupComplete`
   - Then define policy:
     - either allow real inference before warmup,
     - or block with guaranteed queue+replay.

3. **Eliminate placeholder-only non-ready response for first real request**
   - If non-ready, enqueue request + return delivery ack metadata (queued/requestId/eta).
   - Auto-dispatch queued request when ready.

4. **Make warmup non-chat and non-blocking by contract**
   - Warmup response should never appear in chat.
   - But warmup completion should emit a readiness event that flushes pending queue.

5. **Replace single-slot `_pendingWarmupMessage` with bounded FIFO**
   - Avoid silent ignores on second tap.
   - Preserve user intent deterministically.

6. **Add startup observability IDs**
   - Track requestId across Flutter send → native enqueue → dispatch → completion.
   - Add explicit logs for: queued, dropped, replayed, superseded.

7. **Backstop test plan**
   - First message during loading.
   - First message during warmup.
   - Two rapid sends before readiness.
   - Ensure exactly-once replay and visible response without resend.