---
title: HIVE_IMPLEMENTATION_SPEC
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/HIVE_IMPLEMENTATION_SPEC.md
updated: 2026-07-24
---

# Hive Implementation Spec for git-fit

**Date**: 2026-02-14
**Status**: Draft
**Prerequisite**: [TRAINER_AUDIT_DESKTOP_VIABILITY.md](TRAINER_AUDIT_DESKTOP_VIABILITY.md)
**Source Architecture**: EVO Obsidian Vault — Hive MOC + Swarm MOC + 24 Atoms

---

## 0. Problem Statement

iOS kills background tasks after ~30 seconds (`BGProcessingTask` gets slightly more if charging + WiFi, but is unreliable). The trainer's nightly worker (`nightly_federated_worker.dart`) runs 5 sequential tasks that can take 2+ minutes total:

| Task                                      | Time    | iOS Kill Risk |
| ----------------------------------------- | ------- | ------------- |
| Model sync + asset refresh                | 10-30s  | Medium        |
| User LoRA training (100 steps, Metal GPU) | 30-120s | **HIGH**      |
| GU sample extraction                      | 2-5s    | Low           |
| GU delta aggregation                      | 2-5s    | Low           |
| GT delta aggregation (trainer-only)       | 2-5s    | Low           |

Beyond the nightly worker, trainers need to:

- Batch-process 20+ client weekly reports for LoRA ingestion
- Run multi-client analysis ("how are all my clients doing?")
- Monitor approval queues continuously
- Keep federated upload queue draining

**None of these can reliably run in the background on iOS.**

The Hive solves this by making the trainer's **desktop the always-on compute node** for background work, while the phone/tablet stays the in-person session device.

---

## 1. Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│            Trainer's Hive (Single User)                  │
│                                                          │
│  ┌──────────────────────┐  ┌──────────────────────┐     │
│  │  Mac Desktop          │  │  iPhone               │     │
│  │  (Flutter macOS)      │  │  (Flutter iOS)        │     │
│  │                       │  │                       │     │
│  │  BACKGROUND LEASE:    │  │  LIVE LEASE:          │     │
│  │  • Nightly LoRA train │  │  • LAN workout mon.   │     │
│  │  • GT/GU aggregation  │  │  • Real-time Alice    │     │
│  │  • Federated upload   │  │  • In-person chat     │     │
│  │  • Report ingestion   │  │  • Quick approvals    │     │
│  │  • Multi-client batch │  │                       │     │
│  │  • Approval monitoring│  │  DELEGATES BG WORK:   │     │
│  │                       │  │  • "Run LoRA on Mac"  │     │
│  │  ALSO:                │  │  • "Process reports"  │     │
│  │  • Dashboard UI       │  │                       │     │
│  │  • Program builder    │  │                       │     │
│  │  • Analytics          │  │                       │     │
│  └──────────┬───────────┘  └──────────┬───────────┘     │
│             │                          │                  │
│             └──────────┬───────────────┘                  │
│                        │                                  │
│              Hive Sync Layer                              │
│              (LAN mDNS + Supabase Realtime)              │
└────────────────────────┼─────────────────────────────────┘
                         │
                    Supabase DB
                    (shared backend)
```

### Dual-Lease Model

Unlike the Obsidian spec's single execution lease, git-fit uses **two concurrent lease types**:

| Lease Type           | Holder        | Purpose                                                  | Can Transfer?                                            |
| -------------------- | ------------- | -------------------------------------------------------- | -------------------------------------------------------- |
| **Live Lease**       | Phone/Tablet  | In-person sessions, real-time Alice, LAN monitoring      | Yes — transfers to whichever device detects a LAN client |
| **Background Lease** | Desktop (Mac) | Nightly training, aggregation, batch processing, uploads | Yes — falls back to phone if desktop offline             |

**Why dual leases?** A trainer at the gym needs real-time Alice on their phone while their Mac at home runs LoRA training. These are non-conflicting — the Live Lease handles interactive work, the Background Lease handles autonomous work. The Hive safety properties still hold: no duplicate side effects, no concurrent execution of the _same_ task.

---

## 2. Implementation Phases

### Phase 1: Hive Foundation (Weeks 1-2)

**Goal**: Two devices discover each other, establish trust, sync state.

#### 2.1 Hive Discovery Service

**Maps to**: Hive Definition, Hive Pairing Trust Model, Hive Capability Advertisement

**Reuses**: `lan_workout_server.dart` / `lan_workout_client.dart` mDNS pattern via `nsd` package.

**New file**: `flutter_app/lib/core/hive/hive_discovery_service.dart`

```dart
/// Advertises this device on the LAN and discovers other Hive members.
/// Uses mDNS/Bonjour via the `nsd` package (same as LAN workout).
class HiveDiscoveryService {
  static const String _serviceType = '_evo-hive._tcp';

  /// Advertise this device with capability metadata.
  Future<void> advertise({
    required String deviceId,
    required String deviceName,
    required HiveDeviceType deviceType, // phone, tablet, desktop
    required HiveCapabilityProfile capability,
  });

  /// Discover other Hive devices on the LAN.
  Stream<HiveDiscoveredDevice> discover();

  /// Stop advertising and discovery.
  Future<void> dispose();
}
```

**mDNS TXT record** (capability advertisement):

```
deviceId=<uuid>
deviceType=desktop|phone|tablet
modelStatus=warm|cold|none
computeTier=high|med|low
charging=true|false
batteryPct=85
thermalState=nominal|fair|serious|critical
hasLiveLease=false
hasBgLease=true
```

**Why mDNS?** Already proven in the codebase (`_gitfit-workout._tcp`). Works on LAN without internet. Falls back to Supabase Realtime for WAN discovery (trainer at gym, Mac at home).

#### 2.2 Hive Pairing

**Maps to**: Hive Pairing Trust Model, Device Linking via QR, Main Device Root of Trust

**New file**: `flutter_app/lib/core/hive/hive_pairing_service.dart`

```dart
/// Handles QR-based device pairing and trust establishment.
/// Reuses E2E encryption from trainer_client_chat_screen.dart.
class HivePairingService {
  /// Generate a pairing QR code containing:
  /// - deviceId
  /// - ephemeral X25519 public key
  /// - pairing nonce
  /// - Supabase user ID (for verification)
  Future<String> generatePairingCode();

  /// Scan and process a pairing QR code from another device.
  /// Performs mutual authentication:
  /// 1. Verify both devices belong to same Supabase user
  /// 2. Derive shared secret via X25519 key exchange
  /// 3. Store peer device in local Hive registry
  Future<HivePairResult> processPairingCode(String qrData);

  /// Remove a device from the Hive.
  Future<void> unpairDevice(String deviceId);
}
```

**Trust model**:

- Pairing requires both devices to be logged into the **same Supabase account**
- QR code contains an ephemeral X25519 public key for key exchange
- Shared secret derived via ECDH, stored in platform keychain
- All Hive sync messages are encrypted with the shared secret
- Main device (first device) is the root of trust

**Storage**: `flutter_app/lib/core/hive/hive_device_registry.dart`

```dart
/// Persistent registry of paired Hive devices.
/// Stored in SharedPreferences (lightweight, already used everywhere).
class HiveDeviceRegistry {
  Future<List<HivePairedDevice>> getPairedDevices();
  Future<void> addDevice(HivePairedDevice device);
  Future<void> removeDevice(String deviceId);
  Future<HivePairedDevice?> getDevice(String deviceId);
}
```

#### 2.3 Hive Shared State Backbone

**Maps to**: Hive Shared State Backbone, Hive Log Sync, LoRA Artifact Sync

**New file**: `flutter_app/lib/core/hive/hive_state_sync.dart`

The shared state backbone has two transport layers:

**LAN (same network)**: WebSocket via mDNS discovery — low latency, no cloud dependency.

**WAN (different networks)**: Supabase Realtime channel — works when devices are on different networks (trainer at gym, Mac at home).

```dart
/// Synchronizes Hive state across devices.
class HiveStateSyncService {
  /// State that must sync (from Hive Shared State Backbone atom):
  /// - Chat history (already in Supabase)
  /// - Task manager state (SharedPreferences → needs sync)
  /// - LoRA adapter versions (SharedPreferences → needs sync)
  /// - Lease assignments (ephemeral, broadcast only)
  /// - Work ticket status (ephemeral, broadcast only)

  /// Subscribe to state changes from other Hive devices.
  Stream<HiveStateEvent> get stateEvents;

  /// Broadcast a state change to all Hive devices.
  Future<void> broadcast(HiveStateEvent event);
}
```

**What syncs vs. what doesn't**:

| Data                           | Sync?          | Transport                         | Why                                            |
| ------------------------------ | -------------- | --------------------------------- | ---------------------------------------------- |
| Chat messages                  | Already synced | Supabase (existing)               | Already works                                  |
| Workout sessions               | Already synced | Supabase (existing)               | Already works                                  |
| Lease assignments              | Broadcast only | LAN WebSocket / Supabase Realtime | Ephemeral, no persistence needed               |
| Work ticket status             | Broadcast only | LAN WebSocket / Supabase Realtime | Ephemeral                                      |
| LoRA adapter versions          | Sync metadata  | Supabase + file transfer          | Desktop needs to know phone's adapter version  |
| Training samples (SharedPrefs) | Sync on demand | LAN WebSocket (bulk transfer)     | Desktop needs phone's samples for training     |
| Nightly worker results         | Sync metadata  | Supabase Realtime                 | Phone needs to know desktop completed training |

#### 2.4 New Supabase Table: `hive_devices`

```sql
CREATE TABLE IF NOT EXISTS hive_devices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  device_id TEXT NOT NULL,
  device_name TEXT NOT NULL,
  device_type TEXT NOT NULL CHECK (device_type IN ('phone', 'tablet', 'desktop', 'watch')),
  platform TEXT NOT NULL, -- 'ios', 'macos', 'android', 'windows', 'linux'
  has_live_lease BOOLEAN DEFAULT false,
  has_bg_lease BOOLEAN DEFAULT false,
  last_seen_at TIMESTAMPTZ DEFAULT NOW(),
  capability_profile JSONB DEFAULT '{}',
  paired_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE (user_id, device_id)
);

-- RLS: Users can only see/manage their own devices
ALTER TABLE hive_devices ENABLE ROW LEVEL SECURITY;
CREATE POLICY hive_devices_own ON hive_devices
  FOR ALL TO authenticated
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);
```

---

### Phase 2: Lease Protocol & Work Tickets (Weeks 3-4)

**Goal**: Desktop can hold the Background Lease and execute work tickets from the phone.

#### 2.5 Lease Manager

**Maps to**: Execution Lease Rule, Single Executor Guarantee, Lease Transfer Protocol

**New file**: `flutter_app/lib/core/hive/hive_lease_manager.dart`

```dart
/// Manages execution leases across Hive devices.
///
/// Two lease types (git-fit specific, extends Hive spec):
/// - Live Lease: interactive work (Alice chat, LAN monitoring)
/// - Background Lease: autonomous work (training, aggregation, uploads)
///
/// Safety invariant: At most ONE device holds each lease type at any time.
class HiveLeaseManager {
  /// Request a lease. Returns true if granted.
  Future<bool> requestLease(HiveLeaseType type);

  /// Release a lease (e.g., phone going to background).
  Future<void> releaseLease(HiveLeaseType type);

  /// Transfer a lease to a specific device.
  Future<bool> transferLease(HiveLeaseType type, String targetDeviceId);

  /// Current lease state.
  Stream<HiveLeaseState> get leaseState;
}

enum HiveLeaseType { live, background }

class HiveLeaseState {
  final String? liveLeaseHolder;    // deviceId or null
  final String? bgLeaseHolder;      // deviceId or null
  final DateTime lastHeartbeat;
}
```

**Lease acquisition rules**:

| Scenario                                   | Live Lease                  | Background Lease |
| ------------------------------------------ | --------------------------- | ---------------- |
| Only phone exists                          | Phone                       | Phone            |
| Phone + Desktop, phone foregrounded        | Phone                       | Desktop          |
| Phone + Desktop, phone locked/backgrounded | Desktop (auto-transfer)     | Desktop          |
| Phone + Desktop + Tablet at gym            | Phone (LAN client detected) | Desktop          |
| Desktop offline                            | Phone                       | Phone (fallback) |
| Phone offline                              | Desktop                     | Desktop          |

**Heartbeat**: Every 10 seconds via LAN WebSocket or Supabase Realtime. If a lease holder misses 3 heartbeats (30s), the lease is released and other devices can claim it.

**Conflict resolution**: If two devices claim the same lease simultaneously, the device with the lower `device_id` (lexicographic) wins. This is deterministic and requires no coordination.

#### 2.6 Work Ticket System

**Maps to**: Hive Delegated Work Ticket, Hive Bid Protocol, Hive Bid Scoring Rule, Hive Delegation Safety Rule

**New file**: `flutter_app/lib/core/hive/hive_work_ticket.dart`

```dart
/// A bounded unit of work that can be delegated to another Hive device.
///
/// Maps directly to the Hive Delegated Work Ticket atom.
class HiveWorkTicket {
  final String ticketId;
  final String promptId;        // Reference to originating task
  final HiveWorkType workType;  // What kind of work
  final Map<String, dynamic> input;  // Explicit input data
  final Map<String, dynamic> outputSchema;  // Expected output structure
  final int maxTokens;          // Token budget (for inference tasks)
  final Duration maxDuration;   // Time budget
  final Set<String> disallowedActions;  // Safety constraints
  final HiveTicketUrgency urgency;  // interactive vs background
}

enum HiveWorkType {
  // Nightly worker tasks (refactored from callbackDispatcher)
  nightlyModelSync,
  nightlyLoraTraining,
  guSampleExtraction,
  guDeltaAggregation,
  gtDeltaAggregation,
  federatedUpload,

  // Trainer batch tasks
  batchReportIngestion,   // Process N client weekly reports
  multiClientAnalysis,    // Analyze all clients (future Swarm candidate)

  // Inference tasks (future Swarm shards)
  summarization,
  methodDraft,
  riskAssessment,
}

enum HiveTicketUrgency { interactive, background }
```

**New file**: `flutter_app/lib/core/hive/hive_work_dispatcher.dart`

```dart
/// Dispatches work tickets to Hive devices based on capability and lease.
///
/// Simplified bid protocol for git-fit:
/// - Background Lease holder gets all background tickets automatically
/// - Live Lease holder gets all interactive tickets automatically
/// - If no suitable device, ticket is queued locally
/// - Unknown capability fails closed for tickets that require a model tier
/// - Cross-tier fallback uses highest known model tier, then lexicographic
///   device ID as the deterministic tie-break
/// - A dispatched ticket must emit running progress before the acceptance
///   timeout or the dispatcher reassigns to the next deterministic fallback,
///   then queues if no eligible fallback remains
///
/// "Award" currently means deterministic assignment by lease/capability route.
/// Full bid protocol (from Hive Bid Protocol atom) is not implemented yet and
/// remains Phase 3.
class HiveWorkDispatcher {
  /// Submit a work ticket for execution.
  /// Returns a Future that completes when the ticket is done.
  Future<HiveWorkResult> dispatch(HiveWorkTicket ticket);

  /// Cancel a pending or in-progress ticket.
  Future<void> cancel(String ticketId);

  /// Stream of ticket status updates.
  Stream<HiveTicketStatus> get ticketUpdates;
}
```

#### 2.7 Refactor Nightly Worker into Tickets

**Maps to**: Existing `nightly_federated_worker.dart` → ticketed work units

The current `callbackDispatcher` runs everything sequentially in one `BGProcessingTask`. Refactor each step into a standalone function that can be invoked either locally OR via a work ticket:

**Modified file**: `flutter_app/lib/core/background/nightly_federated_worker.dart`

```dart
// BEFORE: monolithic callbackDispatcher runs everything
// AFTER: each task is a standalone function callable by work ticket

/// Each nightly task becomes a HiveExecutableTask.
/// Can run locally (phone) or be dispatched as a work ticket (desktop).
abstract class HiveExecutableTask {
  HiveWorkType get workType;
  Future<Map<String, dynamic>> execute(Map<String, dynamic> input);
}

class NightlyModelSyncTask implements HiveExecutableTask {
  @override HiveWorkType get workType => HiveWorkType.nightlyModelSync;
  @override Future<Map<String, dynamic>> execute(Map<String, dynamic> input) async {
    await runNightlyModelSync();
    return {'status': 'ok'};
  }
}

class NightlyLoraTrainingTask implements HiveExecutableTask {
  @override HiveWorkType get workType => HiveWorkType.nightlyLoraTraining;
  @override Future<Map<String, dynamic>> execute(Map<String, dynamic> input) async {
    final userId = input['userId'] as String;
    await _runUserLoRATraining(userId);
    return {'status': 'ok'};
  }
}

// ... same pattern for GU/GT aggregation, federated upload
```

**Modified file**: `flutter_app/lib/core/background/native_scheduler.dart`

```dart
// When nightly task fires:
// 1. Check if desktop holds Background Lease
// 2. If yes: dispatch tickets to desktop, phone just monitors
// 3. If no: run locally (current behavior, unchanged)

Future<void> executeNightlyTasks() async {
  final leaseManager = HiveLeaseManager.instance;
  final dispatcher = HiveWorkDispatcher.instance;

  if (leaseManager.bgLeaseHolder != thisDeviceId) {
    // Desktop has BG lease — dispatch tickets
    for (final task in nightlyTasks) {
      await dispatcher.dispatch(HiveWorkTicket(
        workType: task.workType,
        input: {'userId': currentUserId},
        maxDuration: const Duration(minutes: 5),
        urgency: HiveTicketUrgency.background,
        disallowedActions: {'tool_execution', 'secret_access'},
      ));
    }
  } else {
    // We hold BG lease — run locally (existing behavior)
    for (final task in nightlyTasks) {
      await task.execute({'userId': currentUserId});
    }
  }
}
```

---

### Phase 3: Desktop App (Weeks 5-6)

**Goal**: Flutter macOS app that joins the Hive and holds the Background Lease.

#### 2.8 Flutter Desktop Target

**New target**: `flutter_app` already supports macOS (Flutter's built-in desktop support). The key differences from iOS:

| Concern              | iOS                            | macOS Desktop                        |
| -------------------- | ------------------------------ | ------------------------------------ |
| Background execution | `BGProcessingTask` (30s limit) | **Unrestricted** — normal process    |
| llama.cpp            | Metal via xcframework          | Metal via xcframework (same GPU API) |
| mDNS                 | `nsd` package                  | `nsd` package (same API)             |
| WebSocket            | `dart:io`                      | `dart:io` (same API)                 |
| Supabase             | `supabase_flutter`             | `supabase_flutter` (same API)        |
| File system          | Sandboxed, App Group           | Sandboxed, but larger storage        |
| Keychain             | iOS Keychain                   | macOS Keychain                       |

**What changes for macOS**:

1. **No `BGProcessingTask`** — the nightly worker runs as a simple `Timer.periodic` or cron-like scheduler. No iOS kill risk.

2. **`NativeScheduler`** already has `PlatformInfo.isMacOS` — extend it:

```dart
// In native_scheduler.dart, the macOS path:
if (platform.isMacOS) {
  // No BGTaskScheduler needed — run directly
  // Schedule via Timer.periodic or platform-native launchd
  await _scheduleDesktopNightlyTask(delay, config);
}
```

3. **`TrainingController.swift`** — the Swift file already exists and uses llama.cpp. For macOS, the same code compiles against the macOS Metal framework instead of iOS Metal. The xcframework likely needs a macOS slice added.

4. **UI** — the desktop app shows the trainer dashboard, program builder, analytics. Reuses existing trainer UI widgets but with desktop-optimized layouts (wider screens, keyboard shortcuts).

#### 2.9 Desktop-Specific Entry Point

**New file**: `flutter_app/lib/main_desktop.dart`

```dart
/// Desktop entry point.
/// Same as main.dart but:
/// - No Workmanager (not needed — no iOS background limits)
/// - Starts HiveDiscoveryService immediately
/// - Claims Background Lease on startup
/// - Runs nightly worker via Timer.periodic
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Skip Workmanager — desktop has no background execution limits
  // Initialize Supabase, Isar, etc. (same as mobile)

  // Start Hive services
  await HiveDiscoveryService.instance.advertise(...);
  await HiveLeaseManager.instance.requestLease(HiveLeaseType.background);

  // Schedule nightly tasks directly (no BGProcessingTask needed)
  _scheduleNightlyWorker();

  runApp(const EvoDesktopApp());
}
```

#### 2.10 Hive Presence UI

**Maps to**: Hive Device Presence Icons, Hive Icon Status Mapping

**New widget**: `flutter_app/lib/core/hive/presentation/hive_presence_indicator.dart`

```dart
/// Shows small device icons in the app bar or chat header.
///
/// Colors (from Hive Icon Status Mapping atom):
/// - Blue: this device holds a lease (active executor)
/// - Blue flashing: Swarm active (Phase 4)
/// - Green: device online + available for work
/// - Red: device online but unavailable (thermal, low battery, no model)
/// - Gray: device offline
class HivePresenceIndicator extends StatelessWidget {
  final List<HivePairedDevice> devices;
  final HiveLeaseState leaseState;
}
```

This widget goes in the `AppBar` of both mobile and desktop apps, giving the trainer visibility into their Hive.

---

### Phase 4: Swarm Parallel Inference (Weeks 7-8, Optional)

**Goal**: Heavy trainer tasks sharded across multiple devices.

**Maps to**: Swarm MOC, Swarm Shard Catalog, Swarm Task Sharding, Swarm Merge Rule

#### 2.11 Swarm Activation

**When to activate** (from Swarm Activation Criteria atom):

- Estimated inference cost exceeds single-device budget
- Parallel execution is safer (thermal/battery)
- Multiple Hive devices are available and capable

**Concrete git-fit triggers**:

- "Analyze all my clients this week" with 10+ clients
- "Generate next week's programs for everyone"
- Batch LoRA training with large sample sets

#### 2.12 Shard Types for git-fit

Mapping the Swarm Shard Catalog to trainer workflows:

| Shard Catalog Type          | git-fit Application                            |
| --------------------------- | ---------------------------------------------- |
| **Summarization Shard**     | Summarize one client's weekly report           |
| **Requirements Extraction** | Extract training goals from client intake form |
| **Method Draft**            | Draft a workout program for one client         |
| **Risk Assessment**         | Flag overtraining or injury risk for a client  |
| **Acceptance Criteria**     | Define "done" for a client's mesocycle         |

**Example**: "Analyze all 20 clients"

```
Lease Holder (Desktop) decides:
  → 20 Summarization Shards (one per client)
  → Dispatch: 10 to desktop, 6 to tablet, 4 to phone
  → Each shard gets: client's weekly report + last 4 weeks context
  → Each shard returns: structured summary (compliance %, intensity trend, flags)
  → Desktop merges 20 summaries into aggregate dashboard
```

#### 2.13 Swarm Work Ticket (extends Hive Work Ticket)

```dart
class SwarmWorkTicket extends HiveWorkTicket {
  final String swarmSessionId;  // Groups related shards
  final int shardIndex;         // Position in shard set
  final int totalShards;        // Total shards in this swarm
  final String outputSchema;    // JSON Schema for structured output
  // Inherits: disallowedActions, maxTokens, maxDuration from parent
}
```

#### 2.14 Merge Coordinator

**Maps to**: Swarm Merge Rule

```dart
/// Runs on the lease holder. Collects shard results and merges.
class SwarmMergeCoordinator {
  /// Wait for all shards to complete (or timeout).
  /// Validate each result against its output schema.
  /// Resolve conflicts (prefer higher confidence).
  /// Remove duplicates.
  /// Return merged result.
  Future<SwarmMergedResult> awaitAndMerge(String swarmSessionId);
}
```

---

## 3. File Change Summary

### New Files

| File                                                      | Phase | Purpose                            |
| --------------------------------------------------------- | ----- | ---------------------------------- |
| `lib/core/hive/hive_discovery_service.dart`               | 1     | mDNS device discovery              |
| `lib/core/hive/hive_pairing_service.dart`                 | 1     | QR-based trust establishment       |
| `lib/core/hive/hive_device_registry.dart`                 | 1     | Persistent paired device storage   |
| `lib/core/hive/hive_state_sync.dart`                      | 1     | Cross-device state synchronization |
| `lib/core/hive/hive_models.dart`                          | 1     | Shared data models                 |
| `lib/core/hive/hive_lease_manager.dart`                   | 2     | Dual-lease protocol                |
| `lib/core/hive/hive_work_ticket.dart`                     | 2     | Work ticket definitions            |
| `lib/core/hive/hive_work_dispatcher.dart`                 | 2     | Ticket dispatch and routing        |
| `lib/core/hive/hive_executable_task.dart`                 | 2     | Task interface for ticketed work   |
| `lib/core/hive/presentation/hive_presence_indicator.dart` | 3     | Device status icons                |
| `lib/core/hive/presentation/hive_settings_screen.dart`    | 3     | Pairing, device management         |
| `lib/main_desktop.dart`                                   | 3     | Desktop entry point                |
| `lib/core/hive/swarm/swarm_coordinator.dart`              | 4     | Parallel inference orchestration   |
| `lib/core/hive/swarm/swarm_merge_coordinator.dart`        | 4     | Result merging                     |
| `lib/core/hive/swarm/swarm_shard_types.dart`              | 4     | Shard type definitions             |
| `supabase/migrations/0XX_create_hive_devices.sql`         | 1     | Hive device registry table         |

### Modified Files

| File                                                         | Phase | Change                                            |
| ------------------------------------------------------------ | ----- | ------------------------------------------------- |
| `core/background/nightly_federated_worker.dart`              | 2     | Refactor tasks into `HiveExecutableTask` units    |
| `core/background/native_scheduler.dart`                      | 2     | Add Hive-aware dispatch (check BG lease holder)   |
| `core/background/native_scheduler.dart`                      | 3     | Add macOS direct scheduling (no BGProcessingTask) |
| `features/home/presentation/home_screen.dart`                | 3     | Add `HivePresenceIndicator` to AppBar             |
| `features/home/presentation/trainer_settings.dart`           | 3     | Add "Hive Devices" section                        |
| `features/chat/domain/trainer_report_ingestion_service.dart` | 2     | Make ingestion callable via work ticket           |
| `features/chat/domain/trainer_report_aggregator.dart`        | 2     | Make aggregation callable via work ticket         |
| `ios/Runner/AppDelegate.swift`                               | 1     | Register Hive method channels                     |
| `macos/Runner/AppDelegate.swift`                             | 3     | macOS-specific Hive initialization                |

### Unchanged (Reused As-Is)

| File                                                  | Why                                       |
| ----------------------------------------------------- | ----------------------------------------- |
| `lan_workout_server.dart` / `lan_workout_client.dart` | Pattern reused, not modified              |
| `trainer_client_chat_screen.dart`                     | E2E encryption pattern reused             |
| All Supabase Edge Functions                           | HTTP-only, no device awareness needed     |
| `trainer_dashboard.dart`                              | UI works on desktop without changes       |
| `trainer_approval_service.dart`                       | Stateless Supabase client, works anywhere |

---

## 4. Safety Properties (from Hive Atoms)

Every implementation decision must preserve these invariants:

| Property                                                     | Atom Source                             | How We Enforce                                                    |
| ------------------------------------------------------------ | --------------------------------------- | ----------------------------------------------------------------- |
| **No concurrent execution of same task**                     | Single Executor Guarantee               | Lease protocol — one holder per lease type                        |
| **No duplicate side effects**                                | Single Executor Guarantee               | Work tickets are idempotent (ticket ID dedup)                     |
| **Delegation is compute-sharing, not authority-sharing**     | Hive Delegation Safety Rule             | Work tickets have `disallowedActions` set                         |
| **Delegated work cannot execute tools**                      | Hive Delegation Safety Rule             | Default: `disallowedActions: {'tool_execution', 'secret_access'}` |
| **Delegated work cannot approve methods or promote Talents** | Hive Delegation Safety Rule             | Enforced in `HiveExecutableTask.execute()`                        |
| **Tickets prevent scope creep**                              | Hive Delegated Work Ticket              | `maxTokens`, `maxDuration`, `outputSchema` enforced               |
| **Shard failure doesn't block Swarm**                        | Swarm Shard Retry Policy                | 1 retry, then partial merge                                       |
| **Pairing required for participation**                       | Hive Pairing Trust Model                | Unpaired devices cannot receive tickets                           |
| **Maintenance mode pauses everything**                       | Hive Security Settings Maintenance Mode | `HiveLeaseManager.enterMaintenanceMode()`                         |

---

## 5. Existing Infrastructure Reuse Map

| Hive/Swarm Concept    | Existing Code                                                   | Reuse Type                                              |
| --------------------- | --------------------------------------------------------------- | ------------------------------------------------------- |
| mDNS discovery        | `lan_workout_server.dart` (nsd package, `_gitfit-workout._tcp`) | **Pattern clone** — new service type `_evo-hive._tcp`   |
| WebSocket streaming   | `LanWorkoutServer._handleTrainerConnection()`                   | **Pattern clone** — new payload types                   |
| E2E encryption        | `trainer_client_chat_screen.dart` (X25519 + ChaCha20)           | **Direct reuse** — same crypto for Hive pairing         |
| Background scheduling | `native_scheduler.dart` + `NativeSchedulerBridge`               | **Extend** — add macOS path, add Hive dispatch          |
| Nightly worker tasks  | `nightly_federated_worker.dart` (5 sequential tasks)            | **Refactor** — extract into `HiveExecutableTask` units  |
| LoRA training         | `TrainingController.swift` (llama.cpp Metal)                    | **Recompile** — add macOS target to xcframework         |
| Device capability     | `TrainingController.checkDeviceConstraints()`                   | **Extend** — add thermal state, compute tier            |
| Supabase Realtime     | Used for workout sessions (polling, not Realtime channels)      | **New usage** — Realtime channels for WAN Hive sync     |
| SharedPreferences     | Used everywhere for local state                                 | **Sync layer** — selective sync via Hive State Backbone |

---

## 6. Migration Path

### For Existing Users (No Hive)

Nothing changes. The phone continues to run the nightly worker via `BGProcessingTask` exactly as it does today. Hive is opt-in.

### For Trainers Who Add a Desktop

1. Install Flutter macOS app
2. Log in with same Supabase account
3. Scan QR code on phone to pair
4. Desktop automatically claims Background Lease
5. Phone's nightly worker detects desktop has BG lease → delegates tickets
6. Desktop runs nightly tasks without iOS time limits
7. Phone shows Hive presence indicator (blue desktop icon = active)

### Rollback

If desktop goes offline, phone detects missed heartbeats after 30s, reclaims Background Lease, and resumes local execution. Seamless fallback.

---

## 7. Open Questions

| Question                                                                       | Impact      | Suggested Resolution                                                                                                                                                                                        |
| ------------------------------------------------------------------------------ | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Should the macOS app be a menu bar app or a full window?                       | UX          | Menu bar for background-only mode, full window when trainer opens dashboard                                                                                                                                 |
| How to handle LoRA adapter file transfer between devices?                      | Phase 2     | LAN WebSocket for large files, Supabase Storage for WAN                                                                                                                                                     |
| ~~Should Swarm shards use the same 1.5B model or allow heterogeneous models?~~ | ~~Phase 4~~ | **RESOLVED**: Heterogeneous models via Alice Lite/Med/Max tiers. See Section 10. Swarm merge coordinator must handle outputs from different tiers.                                                          |
| How to handle trainer with Windows desktop (no Metal)?                         | Phase 3     | CUDA/Vulkan llama.cpp build, or CPU-only fallback. macOS first.                                                                                                                                             |
| Should the Hive support cross-user devices (trainer's Mac ↔ client's phone)?  | Future      | No — Hive is single-user. Trainer-client relationship uses existing Supabase sync.                                                                                                                          |
| Cross-tier LoRA conversion: can a Lite-trained adapter be applied to Max?      | Phase 2     | Training data (JSONL) is tier-agnostic. Each tier trains its own adapter. Shared training data, separate weights. See Section 10.5.                                                                         |
| Should Alice Lite auto-detect when to delegate to Max?                         | Phase 3     | Yes — use prompt complexity heuristics (token count, multi-step reasoning markers). See Section 12.5.                                                                                                       |
| IOPMAssertion vs sandboxed Mac App Store distribution?                         | Phase 3     | `IOPMAssertionCreateWithName` requires `com.apple.security.temporary-exception.iokit-user-client-class` entitlement. May need non-App-Store distribution or fallback to user-managed Energy Saver settings. |
| Should the 3-execution Talent promotion apply to LoRA training?                | Phase 2     | Yes — first 3 runs are supervised (user approves), then auto-promoted. See Section 12.4.                                                                                                                    |

---

## 8. Dependency Summary

### New Packages (None Required)

All required packages already exist in `pubspec.yaml`:

- `nsd` — mDNS discovery (already used for LAN workout)
- `supabase_flutter` — Realtime channels for WAN sync
- `shared_preferences` — Device registry storage
- `workmanager` — iOS background tasks (existing, unchanged)

### Build Changes

- **macOS target**: `flutter create --platforms=macos` (if not already enabled)
- **xcframework**: Add macOS slice to llama.cpp xcframework for Metal inference on Mac
- **Entitlements**: macOS app needs network access, keychain access

---

## 9. Success Criteria

| Metric                                     | Target                                    |
| ------------------------------------------ | ----------------------------------------- |
| Nightly LoRA training completes on desktop | 100% success rate (no iOS kill)           |
| Lease transfer latency (phone → desktop)   | < 5 seconds                               |
| Heartbeat detection of offline device      | < 30 seconds                              |
| Pairing flow (QR scan to paired)           | < 10 seconds                              |
| Fallback to phone when desktop offline     | Automatic, < 30 seconds                   |
| No duplicate task execution across devices | 0 duplicates (ticket ID dedup)            |
| Desktop holds BG lease across app restarts | Persistent via Supabase                   |
| Correct model tier selected per device     | 100% (hardware detection matches tier)    |
| Desktop stays awake during BG lease        | No unexpected sleep during active tickets |

---

## 10. Alice Model Tiers (Lite / Med / Max)

### 10.1 Overview

Three model tiers, all Qwen 2.5 architecture, all trained on the same data, all compatible with the same LoRA adapters via Bring Your Own Cloud adapter sync:

| Tier           | Model         | Params | GGUF Size (Q4_K_M) | Min RAM | Target Devices                                 |
| -------------- | ------------- | ------ | ------------------ | ------- | ---------------------------------------------- |
| **Alice Lite** | Qwen 2.5 1.5B | 1.5B   | ~986 MB            | 3 GB    | iPhone, low-RAM iPad, Apple Watch companion    |
| **Alice Med**  | Qwen 2.5 3B   | 3B     | ~1.8 GB            | 6 GB    | iPad Pro, high-RAM iPhone (15 Pro+), entry Mac |
| **Alice Max**  | Qwen 2.5 7B   | 7B     | ~4.2 GB            | 12 GB   | Mac desktop, Mac laptop, high-end tablets      |

### 10.2 Hardware Detection & Auto-Selection

The downloader picks the right tier automatically. Extends the existing `DeviceTier` enum in `LlamaEngine.swift` and the `_devicePerformanceTier()` method:

**Selection rules** (applied at download time in `AliceAssetDownloadManager`):

```
Available RAM >= 12 GB  →  Alice Max  (7B)
Available RAM >= 6 GB   →  Alice Med  (3B)
Available RAM >= 3 GB   →  Alice Lite (1.5B)
Available RAM < 3 GB    →  Alice Lite (1.5B) + low-end warning
```

On macOS, `ProcessInfo.processInfo.physicalMemory` returns total system RAM (typically 8-96 GB on modern Macs), so desktops will almost always qualify for Alice Max.

On iOS, the same API returns device RAM (4-8 GB on current iPhones), so phones will typically get Lite or Med.

**Override**: User can manually select a tier in Settings (e.g., a trainer with a 16 GB iPad Pro might prefer Max). The downloader respects the override.

### 10.3 Asset Manifest Changes

**Current**: `AliceAssetDownloadManager._assets` has a single hardcoded GGUF entry:

```dart
_AliceAsset(
  name: 'Alice Qwen2.5-1.5B (GGUF)',
  storagePath: 'models/alice-qwen25-1.5b-q4_k_m.gguf',
  relativeTarget: 'AliceAssets/models/alice-qwen25-1.5b-q4_k_m.gguf',
  useStreaming: true,
),
```

**New**: Replace with a tier-aware manifest:

```dart
static _AliceAsset _modelAssetForTier(AliceModelTier tier) {
  switch (tier) {
    case AliceModelTier.lite:
      return _AliceAsset(
        name: 'Alice Lite — Qwen 2.5 1.5B (GGUF)',
        storagePath: 'models/alice-qwen25-1.5b-q4_k_m.gguf',
        relativeTarget: 'AliceAssets/models/alice-qwen25-1.5b-q4_k_m.gguf',
        useStreaming: true,
      );
    case AliceModelTier.med:
      return _AliceAsset(
        name: 'Alice Med — Qwen 2.5 3B (GGUF)',
        storagePath: 'models/alice-qwen25-3b-q4_k_m.gguf',
        relativeTarget: 'AliceAssets/models/alice-qwen25-3b-q4_k_m.gguf',
        useStreaming: true,
      );
    case AliceModelTier.max:
      return _AliceAsset(
        name: 'Alice Max — Qwen 2.5 7B (GGUF)',
        storagePath: 'models/alice-qwen25-7b-q4_k_m.gguf',
        relativeTarget: 'AliceAssets/models/alice-qwen25-7b-q4_k_m.gguf',
        useStreaming: true,
      );
  }
}
```

**Key design**: `LlamaEngine.findFirstGguf()` already scans for any valid `.gguf` file by magic bytes + minimum size — no hardcoded filename. This means swapping tiers requires only downloading the new file and deleting the old one. Zero code changes in the inference path.

### 10.4 LlamaEngine Adaptation per Tier

The existing `LlamaEngine.swift` already queries model architecture at runtime (`llama_model_n_layer`, `llama_model_n_embd`, `llama_model_n_ctx_train`) and calculates safe `n_ctx` and `n_batch` from device memory. This means **no LlamaEngine changes are needed** — the engine auto-adapts to whatever GGUF it loads:

| Parameter             | Lite (1.5B, 28 layers) | Med (3B, 36 layers) | Max (7B, 32 layers) |
| --------------------- | ---------------------- | ------------------- | ------------------- |
| KV cache/token        | ~1 KB                  | ~2 KB               | ~4 KB               |
| n_ctx (8GB device)    | 8192                   | 4096                | 2048                |
| n_ctx (16GB Mac)      | 8192                   | 8192                | 8192                |
| GPU layers (8GB)      | 99 (all)               | 99 (all)            | 32 (all)            |
| GPU layers (16GB Mac) | 99                     | 99                  | 99                  |

### 10.5 LoRA Compatibility Across Tiers

All three tiers are Qwen 2.5 architecture. LoRA adapters target specific layers, so:

- **Same-tier LoRAs**: Direct apply (e.g., Lite LoRA on Lite model) ✓
- **Cross-tier LoRAs**: Requires adapter conversion (different layer counts/dimensions)

**Bring Your Own Cloud (BYOC) adapter sync** ensures all devices have access to the same LoRA library. The adapter metadata includes the target tier:

```json
{
  "adapter_id": "u-lora-2026-02-14",
  "target_tier": "lite",
  "target_architecture": "qwen2.5-1.5b",
  "rank": 8,
  "checksum": "sha256:..."
}
```

Devices only load adapters matching their active tier. Cross-tier training (e.g., training on Lite data but applying to Max) is a future consideration — the training data (JSONL) is tier-agnostic, only the adapter weights are tier-specific.

---

## 11. Desktop Sleep Prevention

### 11.1 The Problem

macOS will sleep the display and eventually the system if idle. A trainer's desktop holding the Background Lease must stay awake to:

- Complete nightly LoRA training (30-120s of GPU compute)
- Process work tickets from the phone
- Drain the federated upload queue
- Respond to heartbeat pings

### 11.2 Setup Instructions (User-Facing)

The desktop app should guide the trainer through sleep prevention on first launch:

**Option A: App-Managed (Recommended)**

The Flutter macOS app uses `IOPMAssertionCreateWithName` (via platform channel) to prevent sleep while holding the Background Lease:

```swift
// macOS native channel: evo/power_management
import IOKit.pwr_mgt

var assertionID: IOPMAssertionID = 0

func preventSleep(reason: String) -> Bool {
    let result = IOPMAssertionCreateWithName(
        kIOPMAssertPreventUserIdleSystemSleep as CFString,
        IOPMAssertionLevel(kIOPMAssertionLevelOn),
        reason as CFString,
        &assertionID
    )
    return result == kIOReturnSuccess
}

func allowSleep() {
    IOPMAssertionRelease(assertionID)
}
```

**Behavior**:

- When the app acquires the Background Lease → `preventSleep("Alice Background Lease active")`
- When the app releases the Background Lease → `allowSleep()`
- When all work tickets are complete and no pending work → `allowSleep()`
- Display can still sleep (screen off) — only system sleep is prevented

**Option B: System Settings Fallback**

If the app can't manage power assertions (sandboxing restrictions), guide the user:

1. **System Settings → Energy Saver → Prevent automatic sleeping when the display is off** ✓
2. **System Settings → Lock Screen → Turn display off → Never** (optional, saves energy if set to a timeout)
3. **Enable Wake for network access** ✓ (allows Alice to wake the Mac when the phone sends a work ticket via WAN)

### 11.3 Wake-on-LAN for Remote Tickets

When the trainer is at the gym (phone has Live Lease) and sends a work ticket to the desktop (Background Lease holder) but the Mac has gone to sleep:

1. Phone sends a Wake-on-LAN magic packet to the Mac's known MAC address (stored during Hive pairing)
2. Mac wakes, reconnects to Hive, picks up the work ticket
3. Mac completes the ticket, sends result back via Supabase Realtime
4. Mac returns to sleep after idle timeout (if no more pending tickets)

**Requires**: "Wake for network access" enabled in macOS Energy Saver settings. The Hive pairing flow should check this and prompt the user if disabled.

### 11.4 Alice-Initiated Wake

When Alice on the phone determines a task should be delegated to the desktop:

1. Alice checks Hive lease state — desktop holds BG lease but last heartbeat is stale
2. Alice sends Wake-on-LAN packet
3. Alice queues the work ticket in Supabase (persistent, survives phone backgrounding)
4. Desktop wakes, syncs with Supabase, picks up queued ticket
5. Desktop executes and posts result to Supabase
6. Phone picks up result on next foreground or via push notification

This means Alice can effectively "wake up her own desktop" to run heavy tasks — the user doesn't need to manually open the Mac.

---

## 12. Hive-Aware Model Routing

### 12.1 The Core Idea

With multiple model tiers across devices, the Hive can route tasks to the **most capable device** for that task type. This is the intersection of the Hive Bid Protocol and the multi-model architecture:

```
Trainer's Hive:
  iPhone (Alice Lite, 1.5B)  — fast, always available, limited reasoning
  iPad Pro (Alice Med, 3B)   — balanced, gym companion
  Mac Mini (Alice Max, 7B)   — powerful, always-on, deep reasoning
```

### 12.2 Routing Rules

| Task Type                          | Best Device               | Why                                     |
| ---------------------------------- | ------------------------- | --------------------------------------- |
| Quick chat ("what's my next set?") | Whichever has Live Lease  | Latency matters, any tier handles it    |
| Workout coaching (real-time)       | Phone/Tablet (Live Lease) | Needs LAN proximity to client           |
| Weekly report summarization        | Mac (Alice Max)           | Better reasoning = better summaries     |
| Multi-client batch analysis        | Mac (Alice Max)           | Heavy inference, no time limit          |
| LoRA training                      | Mac (Alice Max)           | GPU compute, no iOS kill risk           |
| Program generation                 | Mac (Alice Max)           | Complex reasoning benefits from 7B      |
| Nutrition target calculation       | Any                       | Simple math, any tier handles it        |
| Method proposal (Talent system)    | Mac (Alice Max) preferred | Better method quality from larger model |

### 12.3 Capability Advertisement Extension

The Hive Capability Advertisement (Section 2.1) now includes model tier:

```
deviceId=<uuid>
deviceType=desktop
modelTier=max          # NEW: lite, med, max, or none
modelParams=7B         # NEW: human-readable param count
modelStatus=warm
computeTier=high
charging=true
batteryPct=100
thermalState=nominal
hasLiveLease=false
hasBgLease=true
```

The lease holder uses `modelTier` in bid scoring (Section 2.6) to route complex tasks to the most capable device.

### 12.4 LoRA Training as a Talent

LoRA training is a perfect candidate for the Talent system:

**Why a Talent?**

- It's a multi-step process (prepare data → write JSONL → run training → verify adapter → sync)
- It has clear success/failure criteria
- It benefits from the 3-execution promotion rule (first few runs may need tuning)
- It should eventually run autonomously without re-approval

**Method definition**:

```yaml
talent_candidate: "nightly_lora_training"
steps: 1. Collect training samples from last 7 days (SharedPreferences)
  2. Write samples to JSONL file
  3. Validate JSONL format and sample count (min 3 samples)
  4. Run llama.cpp training (llama_opt_init → llama_opt_epoch × N)
  5. Verify adapter checksum and size
  6. Sync adapter metadata to Hive (Supabase)
  7. Clean up temporary JSONL file
allowed_tools:
  - file_write (JSONL only)
  - training_controller (native Metal/CUDA)
  - supabase_upload (adapter metadata only)
disallowed_tools:
  - chat_send
  - approval_queue
  - secret_access
max_duration: 300s # 5 minutes
```

**Hive routing for LoRA training**:

1. Nightly scheduler fires on phone (via `BGProcessingTask`)
2. Phone checks: "Does any Hive device have Alice Max + BG lease?"
3. If yes → dispatch LoRA training work ticket to Mac
4. Mac executes the Talent (all 7 steps)
5. Mac syncs adapter metadata to Supabase
6. Phone picks up new adapter version on next sync
7. If no Mac available → phone runs training locally with Alice Lite (existing behavior, iOS kill risk accepted)

**Why the Mac is better for training**:

- Alice Max (7B) produces higher-quality training gradients than Lite (1.5B)
- macOS has no background execution time limit
- Mac GPU (M1/M2/M3/M4) has more compute and thermal headroom
- Training can run for minutes without risk of iOS killing the process

### 12.5 Cross-Tier Inference Delegation

When the phone (Alice Lite) encounters a task that would benefit from deeper reasoning:

1. Alice Lite on phone recognizes the task is complex (e.g., "design a 12-week periodization program")
2. Phone checks Hive: Mac has Alice Max, is online, holds BG lease
3. Phone dispatches an inference work ticket to Mac:
   ```
   HiveWorkTicket(
     workType: HiveWorkType.methodDraft,
     input: { 'prompt': '...', 'context': '...' },
     maxTokens: 4096,
     maxDuration: Duration(seconds: 60),
     urgency: HiveTicketUrgency.interactive,
     disallowedActions: {'tool_execution', 'secret_access'},
   )
   ```
4. Mac runs inference with Alice Max (7B) — better reasoning, longer context
5. Mac returns the result to phone via Hive sync
6. Phone presents the result to the user as if Alice generated it locally

**User experience**: The trainer asks Alice a complex question on their phone. Alice says "Let me think about this on your Mac..." (optional transparency). A few seconds later, the answer appears — generated by the 7B model on the desktop but presented seamlessly on the phone.

### 12.6 Model Tier in Hive Presence UI

The Hive Presence Indicator (Section 2.10) should show model tier:

```
[🔵 Mac Mini — Alice Max (7B)]  [🟢 iPhone — Alice Lite (1.5B)]  [⚫ iPad — offline]
```

This gives the trainer visibility into which device has the most capable Alice, and where heavy work will be routed.

## Related

^[source-materials/mirrors/doctrine/HIVE_IMPLEMENTATION_SPEC.md]
