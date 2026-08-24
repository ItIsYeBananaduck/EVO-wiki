---
title: Hive Packaging + Supabase Extraction Plan
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Hive Packaging + Supabase Extraction Plan.md"]
updated: 2026-07-24
---

# Hive Packaging + Supabase Extraction Plan
Purpose

This note defines the current state of Hive in the repo, why packaging is incomplete, what should live in shared monorepo packages versus app-layer code, and how to safely remove Supabase from Hive control-plane responsibilities without breaking the active implementation.

IMPORTANT: This note is internal only. Mark as NOT for book.

Context

The repo currently contains two competing Hive directions.

Direction one is a Supabase-centered Hive control plane:

- backend device registry
- lease persistence / shared state assumptions
- dual-lease worldview
- desktop background executor framing

Direction two is a Cloudflare-signaled peer-to-peer model:

- Cloudflare Worker signaling
- WebRTC DataChannel transport
- direct peer coordination
- local Hive membership and peer presence

The second direction is closer to the intended EVO architecture.

Architectural Correction

Hive must be treated as a user-owned, local-first delegation layer.

Supabase must not act as the Hive control plane.

That means Supabase cannot be authoritative for:

- Hive node membership
- node presence
- lease ownership
- node topology
- canonical delegation state

Cloud infrastructure may assist with signaling or transport, but it must not become the semantic owner of the Hive.

Cloudflare is currently the better fit for Hive node chat and trainer-client peer signaling because the repo already contains a Cloudflare Worker signaling path and WebRTC transport direction.

Current Repo Findings

What is already packaged

The repo already has a shared Hive package containing core nouns and orchestration primitives.

Files:

- packages/hive/lib/evo_hive.dart
- packages/hive/lib/src/hive_orchestration.dart
- packages/hive/lib/src/hive_node.dart
- packages/hive/lib/src/hive_task_execution.dart
- packages/hive/lib/src/hive_event.dart
- packages/hive/lib/src/hive_presence_state.dart
- packages/hive/lib/src/hive_registry.dart

What is still stranded in app-layer code

Most live runtime behavior is still in flutter_app/lib/core/hive.

Files:

- flutter_app/lib/core/hive/hive_lease_manager.dart
- flutter_app/lib/core/hive/hive_state_sync.dart
- flutter_app/lib/core/hive/hive_mailbox_sync.dart
- flutter_app/lib/core/hive/hive_connection_manager.dart
- flutter_app/lib/core/hive/hive_inference_router.dart
- flutter_app/lib/core/hive/hive_webrtc_transport.dart
- flutter_app/lib/core/hive/hive_alice_mesh.dart

Trainer/client signaling is also still app-level:

- flutter_app/lib/features/chat/domain/p2p_signaling_client.dart

Why Packaging Is Incomplete

Reason one

The repo extracted the Hive nouns but not the Hive behaviors.

Shared contracts and orchestration primitives were moved into packages, but concrete runtime services remained inside app code.

Reason two

The un-packaged pieces are tightly coupled to Flutter runtime and product behavior.

They currently mix:

- shared Hive logic
- transport implementation
- Flutter singletons
- local stores
- app lifecycle behavior
- older training-specific assumptions

Reason three

The architecture is still split between two incompatible control-plane assumptions:

- Supabase-backed state sync / device registry
- Cloudflare-signaled peer-to-peer communication

Until that conflict is resolved, packaging remains muddy.

Reason four

Some pieces are still conceptually unresolved.

Open questions include:

- whether delegation should be represented as a root/runtime behavior or as a Talent-governed path
- whether Alice mesh should remain a shared runtime primitive or be narrowed into a more targeted node chat protocol
- whether mailbox sync is a core Hive transport or only an optional WAN fallback

Packaging Matrix

Keep in shared package now

These belong in packages/hive or equivalent shared monorepo space now:

- node model
- event model
- task execution model
- presence state model
- task claim / ownership primitives
- orchestration contracts
- transport interfaces
- peer identity / trust contracts
- control message schemas

Move to shared package after refactor

- hive_connection_manager
- hive_webrtc_transport
- hive_alice_mesh
- hive_state_sync
- hive_mailbox_sync
- p2p_signaling_client

Likely app-specific

- UI surfaces
- task panels
- training chat UI
- device settings

Supabase Removal Goal

Remove Supabase entirely from Hive control-plane responsibilities while preserving working device coordination.

Supabase must no longer be used for:

- hive device registry
- authoritative lease state
- canonical node presence
- semantic Hive shared state

Safe Extraction Sequence

Phase 1 — Freeze

- stop new Supabase Hive logic

Phase 2 — Local-first control plane

- HiveStore + pairing define membership
- heartbeats define presence
- P2P defines delegation

Phase 3 — Split packages

- move contracts + protocols into packages
- keep adapters in app layer

Phase 4 — Remove Supabase

- delete hive_devices usage
- remove registry assumptions

Phase 5 — Replace lease model

- remove dual lease
- move to task-scoped ownership

Conclusion

This is a platform correction, not a rewrite.

We preserve:

- Cloudflare signaling
- WebRTC P2P
- Hive package core

We remove:

- Supabase control-plane usage
- dual-lease assumptions

We restructure:

- shared runtime vs app wiring

Source

Internal GitHub audit

## Related
