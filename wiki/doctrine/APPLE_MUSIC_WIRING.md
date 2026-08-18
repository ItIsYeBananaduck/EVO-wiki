---
title: APPLE_MUSIC_WIRING
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/APPLE_MUSIC_WIRING.md"]
updated: 2026-07-24
---

# Apple Music Integration - How It's Wired Up

## Current State: 🟡 Scaffolded but Not Implemented

The app has a complete **architecture** for Apple Music integration, but the actual **native iOS implementation** is missing. All the Flutter-side code is ready and waiting for the native bridge.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│ Live Workout Screen (Flutter)                               │
│  - Listens to playback state stream                         │
│  - Calls play/pause/next/startSession                       │
│  - Updates UI with track info                               │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ↓
┌─────────────────────────────────────────────────────────────┐
│ AppleMusicStrainSyncBackend (Dart)                          │
│  - Abstract interface implementation                        │
│  - Methods: play(), pause(), next(), startSession()         │
│  - Stream: playbackStateStream                              │
│  ⚠️  ALL METHODS ARE EMPTY STUBS                            │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ↓ MISSING LINK ❌
                 │
┌─────────────────────────────────────────────────────────────┐
│ iOS MusicKit Integration (Swift) - NOT YET IMPLEMENTED      │
│  - MethodChannel to receive commands from Flutter           │
│  - MusicKit API to control Apple Music                      │
│  - EventChannel to stream playback state                    │
└─────────────────────────────────────────────────────────────┘
```

---

## Where It's Used in the App

### 1. **Live Workout Screen** (`lib/features/home/presentation/live_workout_screen.dart`)

#### Lines 475-481: Start Music Session

```dart
appleMusicStrainSyncBackend.startSession(
  userId: widget.user.id,
  sessionId: _sessionId,
  mode: mode,  // StrainSyncMode (push/recovery/steady/auto)
  tempoBand: band,  // BPM range for workout intensity
  expectedDuration: null,
);
```

**Purpose:** Starts an AI-driven music session that matches workout intensity

#### Lines 905-907: Listen to Playback State

```dart
_musicSub = appleMusicStrainSyncBackend.playbackStateStream.listen(
  _handlePlaybackState,
);
```

**Purpose:** Real-time updates of current track, play/pause status, position

#### Lines 2263-2276: Playback Controls

```dart
// Play button
appleMusicStrainSyncBackend.play();

// Next track button
appleMusicStrainSyncBackend.next();
```

**Purpose:** User controls for music playback during workout

#### Lines 2331-2349: Handle Playback State Updates

```dart
void _handlePlaybackState(StrainSyncPlaybackState state) {
  final String key = 'apple_music:${state.trackId}';
  setState(() {
    _currentTrackKey = key;
    _currentTrackName = state.trackName;
    _currentArtistName = state.artistName;
    _currentPlaylistId = state.playlistId;
    _currentTrackPositionSeconds = state.positionSeconds;
    _currentTrackDurationSeconds = state.durationSeconds;
    _currentTrackIsPlaying = state.isPlaying;
  });
}
```

**Purpose:** Updates UI with current track info, logs music/intensity correlation

### 2. **Music Backend** (`lib/features/intensity/domain/strain_sync_music_backend.dart`)

#### Lines 61-113: Backend Implementation (EMPTY!)

```dart
class AppleMusicStrainSyncBackend implements StrainSyncMusicBackend {
  @override
  Future<bool> isAvailable() async {
    return true;  // ⚠️ Should check if MusicKit is authorized
  }

  @override
  Future<void> startSession({...}) async {}  // ⚠️ EMPTY

  @override
  Future<void> play() async {}  // ⚠️ EMPTY

  @override
  Future<void> pause() async {}  // ⚠️ EMPTY

  @override
  Future<void> next() async {}  // ⚠️ EMPTY
}
```

---

## What Needs to Be Implemented

### Step 1: Add iOS Permissions & Capabilities

#### `ios/Runner/Info.plist`

```xml
<key>NSAppleMusicUsageDescription</key>
<string>EVOtraining syncs workout intensity with your music to optimize performance and recovery.</string>
```

#### Xcode Project Capabilities

- ✅ Enable **MusicKit** capability in Xcode
- ✅ Sign up for Apple Developer Program (required for MusicKit)

### Step 2: Create Swift MethodChannel Bridge

#### `ios/Runner/AppleMusicBridge.swift` (NEW FILE)

```swift
import Flutter
import MusicKit

class AppleMusicBridge {
    private let channel: FlutterMethodChannel
    private var player: ApplicationMusicPlayer?
    private var eventSink: FlutterEventSink?

    init(binaryMessenger: FlutterBinaryMessenger) {
        channel = FlutterMethodChannel(
            name: "evo/apple_music",
            binaryMessenger: binaryMessenger
        )
        setupMethodChannel()
        setupEventChannel(binaryMessenger: binaryMessenger)
    }

    private func setupMethodChannel() {
        channel.setMethodCallHandler { [weak self] (call, result) in
            switch call.method {
            case "isAvailable":
                self?.handleIsAvailable(result: result)
            case "requestAuthorization":
                self?.handleRequestAuthorization(result: result)
            case "play":
                self?.handlePlay(result: result)
            case "pause":
                self?.handlePause(result: result)
            case "next":
                self?.handleNext(result: result)
            case "startSession":
                self?.handleStartSession(args: call.arguments, result: result)
            default:
                result(FlutterMethodNotImplemented)
            }
        }
    }

    private func setupEventChannel(binaryMessenger: FlutterBinaryMessenger) {
        let eventChannel = FlutterEventChannel(
            name: "evo/apple_music_events",
            binaryMessenger: binaryMessenger
        )
        eventChannel.setStreamHandler(self)
    }

    // Implementation methods...
}
```

### Step 3: Update Dart Backend to Use MethodChannel

#### `lib/features/intensity/domain/strain_sync_music_backend.dart`

```dart
class AppleMusicStrainSyncBackend implements StrainSyncMusicBackend {
  static const MethodChannel _channel = MethodChannel('evo/apple_music');
  static const EventChannel _eventChannel = EventChannel('evo/apple_music_events');

  @override
  Future<bool> isAvailable() async {
    try {
      return await _channel.invokeMethod<bool>('isAvailable') ?? false;
    } catch (e) {
      return false;
    }
  }

  @override
  Future<void> play() async {
    await _channel.invokeMethod('play');
  }

  @override
  Future<void> next() async {
    await _channel.invokeMethod('next');
  }

  @override
  Future<void> startSession({...}) async {
    await _channel.invokeMethod('startSession', {
      'userId': userId,
      'mode': mode?.name,
      'minBpm': tempoBand?.minBpm,
      'maxBpm': tempoBand?.maxBpm,
    });
  }

  @override
  Stream<StrainSyncPlaybackState> get playbackStateStream {
    return _eventChannel.receiveBroadcastStream().map((event) {
      return StrainSyncPlaybackState(
        trackId: event['trackId'],
        trackName: event['trackName'],
        artistName: event['artistName'],
        isPlaying: event['isPlaying'],
        positionSeconds: event['positionSeconds'],
        durationSeconds: event['durationSeconds'],
      );
    });
  }
}
```

### Step 4: Register Bridge in AppDelegate

#### `ios/Runner/AppDelegate.swift`

```swift
@main
@objc class AppDelegate: FlutterAppDelegate {
  private var appleMusicBridge: AppleMusicBridge?

  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    let controller = window?.rootViewController as! FlutterViewController

    // Initialize Apple Music bridge
    appleMusicBridge = AppleMusicBridge(binaryMessenger: controller.binaryMessenger)

    // ... existing code ...
  }
}
```

---

## Data Flow

### When User Starts Workout:

1. **Flutter:** `appleMusicStrainSyncBackend.startSession(mode: push, tempoBand: 120-140)`
2. **Swift:** Receives via MethodChannel
3. **MusicKit:** Queries Apple Music for tracks matching BPM range
4. **MusicKit:** Creates playlist and starts playback
5. **Swift:** Streams playback state via EventChannel
6. **Flutter:** Updates UI with track name, artist, progress bar

### When User Taps Play/Next:

1. **Flutter:** `appleMusicStrainSyncBackend.play()` or `.next()`
2. **Swift:** Receives via MethodChannel
3. **MusicKit:** Controls ApplicationMusicPlayer
4. **Swift:** Emits new playback state
5. **Flutter:** UI updates automatically

---

## Feature Logic Already Implemented

### ✅ Strain Sync Modes

- **Push:** High-energy tracks (high BPM)
- **Recovery:** Calming tracks (low BPM)
- **Steady:** Moderate intensity
- **Auto:** AI-driven based on workout performance

### ✅ Music Intelligence

- Tracks user's thumbs up/down on songs
- Correlates music with workout performance
- Learns which songs boost performance
- Auto-adjusts tempo to workout intensity

### ✅ Access Control

Only Pro/Trainer/Admin users can use StrainSync:

```dart
bool get _canUseStrainSync =>
    widget.user.isPro || widget.user.isTrainer || widget.user.isAdmin;
```

### ✅ UI Components

- Play/pause/next buttons (lines 2258-2277)
- Track name display
- Playback progress slider
- Thumbs up/down for track rating

---

## Summary

**Status:** 📐 Architecture complete, native implementation missing

**What's Working:**

- ✅ Flutter UI and controls
- ✅ Playback state handling
- ✅ Music intelligence and logging
- ✅ Workout intensity correlation

**What's Missing:**

- ❌ iOS MusicKit integration
- ❌ MethodChannel bridge
- ❌ EventChannel for state streaming
- ❌ Apple Music authorization flow

**Effort Required:** ~2-4 hours to implement the native iOS bridge

**Key Files to Create/Modify:**

1. NEW: `ios/Runner/AppleMusicBridge.swift`
2. UPDATE: `ios/Runner/AppDelegate.swift`
3. UPDATE: `lib/features/intensity/domain/strain_sync_music_backend.dart`
4. UPDATE: `ios/Runner/Info.plist`
5. UPDATE: Xcode project capabilities

---

Ready to implement when you are! 🎵

## Related

^[{src_rel}]
