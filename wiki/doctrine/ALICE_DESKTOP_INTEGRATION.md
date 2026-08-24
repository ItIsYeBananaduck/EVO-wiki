---
title: ALICE_DESKTOP_INTEGRATION
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/ALICE_DESKTOP_INTEGRATION.md
updated: 2026-07-24
---

# Alice Chat Desktop Integration

## Overview

This document describes the integration of Alice chat functionality from the mobile app into the desktop trainer workstation.

## Changes Made

### 1. Flutter/Dart Layer

#### TrainerDesktopView (`lib/features/trainer/presentation/trainer_desktop_view.dart`)

- **Added imports**: `AliceChatScreen`, `AppUser`, `Supabase`
- **Added state**: `AppUser? _currentUser` to track authenticated user
- **Added method**: `_loadCurrentUser()` to fetch user profile from Supabase
- **Wired button**: "Chat with Alice" button now opens `AliceChatScreen` when clicked
- **User validation**: Button is disabled until user is loaded

#### Changes:

```dart
// Added to state
AppUser? _currentUser;

// Added to initState
_loadCurrentUser();

// Button now navigates to Alice chat
ElevatedButton.icon(
  onPressed: _currentUser == null ? null : () {
    Navigator.of(context).push(
      MaterialPageRoute(
        builder: (_) => AliceChatScreen(
          user: _currentUser!,
        ),
      ),
    );
  },
  icon: const Icon(Icons.chat),
  label: const Text('Chat with Alice'),
  ...
)
```

### 2. Native macOS Layer

#### Swift Files Copied from iOS to macOS

The following Swift files were copied from `ios/Runner/` to `macos/Runner/`:

**Core Alice Inference:**

- `AliceInferenceManager.swift` - Main inference coordinator
- `LlamaEngine.swift` - llama.cpp inference engine
- `LlamaEngine+Skills.swift` - Skills/Talents extension
- `PromptBuilder.swift` - ChatML prompt construction
- `PromptCompressor.swift` - Context window management
- `ToolCallingFramework.swift` - Tool calling support

**Talent System:**

- `TalentRegistry.swift` - Pre-built Talent definitions
- `TalentRoutingEngine.swift` - Deterministic routing
- `TalentPromptBuilder.swift` - Per-step prompt generation
- `TaskListExecutor.swift` - Task execution engine

**Supporting Files:**

- `ModelArchitecture.swift` - Model metadata
- `EffectiveConfig.swift` - Configuration management
- `CrashLogger.swift` - Error logging
- `TrainingController.swift` - LoRA training
- `ContextResizer.swift` - Context window utilities

#### AppDelegate Updates (`macos/Runner/AppDelegate.swift`)

- **Added**: `AliceInferenceManager.shared.register(with: messenger)` to register Alice inference channel

## What Works

✅ **Alice chat screen** - Full chat UI with all mobile features
✅ **Navigation** - "Chat with Alice" button opens chat screen
✅ **User authentication** - Loads user profile from Supabase
✅ **Swift files** - All necessary native code copied to macOS
✅ **Plugin registration** - AliceInferenceManager registered in AppDelegate

## What Still Needs to Be Done

### Critical (Required for Alice to work)

1. **Add Swift files to Xcode project**
   - Open `macos/Runner.xcworkspace` in Xcode
   - Add all copied `.swift` files to the Runner target
   - Ensure they're in the "Compile Sources" build phase
   - Files to add:
     - AliceInferenceManager.swift
     - LlamaEngine.swift
     - LlamaEngine+Skills.swift
     - PromptBuilder.swift
     - PromptCompressor.swift
     - ToolCallingFramework.swift
     - TalentRegistry.swift
     - TalentRoutingEngine.swift
     - TalentPromptBuilder.swift
     - TaskListExecutor.swift
     - ModelArchitecture.swift
     - EffectiveConfig.swift
     - CrashLogger.swift
     - TrainingController.swift
     - ContextResizer.swift

2. **Add llama.cpp framework to macOS**
   - Build or download llama.cpp xcframework for macOS
   - Add to `macos/Frameworks/` directory
   - Link in Xcode project settings
   - Update `macos/Runner/Info.plist` if needed

3. **Copy alice_capability_map.json to macOS bundle**
   - Ensure `flutter_app/assets/alice_capability_map.json` is in macOS bundle
   - Update `macos/Runner/Info.plist` to include asset catalog

### Optional (Enhanced Features)

4. **Desktop-specific UI optimizations**
   - Larger chat window for desktop screens
   - Keyboard shortcuts (Cmd+K to open chat, etc.)
   - Multi-window support (chat in separate window)

5. **Desktop-specific features**
   - File drag-and-drop for CSV imports
   - Better file picker integration
   - Desktop notifications for Alice responses

6. **Performance optimizations**
   - Utilize desktop GPU for faster inference
   - Larger context windows on desktop (more RAM available)
   - Background inference while app is minimized

## Testing Checklist

Once the above steps are completed, test the following:

- [ ] Open desktop app and click "Chat with Alice"
- [ ] Alice chat screen opens without errors
- [ ] Send a message to Alice
- [ ] Alice responds with inference (not just echo)
- [ ] Tool calling works (e.g., "update my nutrition targets")
- [ ] File imports work (CSV plans, body composition scans)
- [ ] TTS works (Alice speaks responses)
- [ ] Chat history persists between sessions
- [ ] Multiple chat sessions can be created

## Architecture Notes

### Shared Code

The Alice chat implementation is **fully shared** between mobile and desktop:

- Same Dart UI code (`AliceChatScreen`)
- Same domain logic (services, models)
- Same Swift inference code (copied from iOS)

### Platform Differences

The only platform-specific code is:

- **iOS**: Uses BGTaskScheduler for background tasks
- **macOS**: Uses Timer.periodic (no 30s kill limit)
- **macOS**: Different file paths for model storage
- **macOS**: CoreAudio instead of AVAudioSession

### Model Storage

Desktop uses the same `SharedModelStore` as mobile:

- Models stored in `~/Library/Application Support/EVOtraining/AliceAssets/models/`
- Same download mechanism via `AliceAssetDownloadManager`
- Same GGUF model files (alice-qwen25-1.5b-q4_k_m.gguf)

## Next Steps

1. **Immediate**: Add Swift files to Xcode project (see Critical #1 above)
2. **Immediate**: Add llama.cpp framework (see Critical #2 above)
3. **Test**: Verify Alice chat works on desktop
4. **Polish**: Add desktop-specific UI enhancements
5. **Document**: Update user-facing docs with desktop chat instructions

## Related Files

- Mobile chat: `lib/features/alice/presentation/alice_chat_screen.dart`
- Desktop view: `lib/features/trainer/presentation/trainer_desktop_view.dart`
- iOS native: `ios/Runner/AliceInferenceManager.swift`
- macOS native: `macos/Runner/AliceInferenceManager.swift`
- AppDelegate: `macos/Runner/AppDelegate.swift`
^[source-materials/mirrors/doctrine/ALICE_DESKTOP_INTEGRATION.md]
