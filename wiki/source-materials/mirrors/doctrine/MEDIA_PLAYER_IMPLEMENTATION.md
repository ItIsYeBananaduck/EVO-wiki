---
title: MEDIA_PLAYER_IMPLEMENTATION
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/MEDIA_PLAYER_IMPLEMENTATION.md"]
updated: 2026-07-24
---

# Media Player Implementation

## Overview

Added a reusable media player widget to the app that can be used for video workouts, tutorials, and other video content.

## Components

### 1. MediaPlayerWidget

**Location**: `lib/core/widgets/media_player_widget.dart`

**Features**:

- Video playback with network URLs
- Custom controls (play/pause, seek, fullscreen)
- Progress bar with time display
- Auto-play option
- Loop support
- Placeholder while loading
- Callbacks for player ready and completion events

**Usage**:

```dart
MediaPlayerWidget(
  videoUrl: 'https://example.com/video.mp4',
  autoPlay: false,
  showControls: true,
  allowFullscreen: true,
  loop: false,
  onPlayerReady: () => print('Player ready'),
  onPlaybackComplete: () => print('Video finished'),
)
```

### 2. VideoThumbnail

**Location**: `lib/core/widgets/media_player_widget.dart`

**Features**:

- Video thumbnail with play button overlay
- Duration badge
- Title overlay
- Placeholder image support
- Tap-to-play functionality

**Usage**:

```dart
VideoThumbnail(
  videoUrl: 'https://example.com/video.mp4',
  title: 'Workout Tutorial',
  duration: Duration(minutes: 5),
  thumbnailUrl: 'https://example.com/thumb.jpg',
  onTap: () => playVideo(),
)
```

### 3. MediaPlayerDemoScreen

**Location**: `lib/features/media/presentation/media_player_demo_screen.dart`

**Features**:

- Demo screen showcasing media player capabilities
- Video library grid layout
- Sample videos with metadata
- Player view with video info
- Navigation between list and player

**Access**: Available through Alice chat menu (⋮) → "Media Player Demo"

## Integration Points

### Current State

- ✅ Media player widget ready for use
- ✅ Demo screen accessible via Alice chat
- ✅ Video player dependency added
- ✅ Sample videos using public test URLs

### Future Integration Areas

#### 1. Workout Plans

```dart
// In workout plan display
VideoThumbnail(
  videoUrl: exercise.videoUrl,
  title: exercise.name,
  duration: exercise.videoDuration,
  onTap: () => playExerciseVideo(exercise),
)
```

#### 2. Trainer Upload Flow

```dart
// In TrainerMarketplaceUpload
// Add video upload section
FilePickerResult? result = await FilePicker.platform.pickFiles(
  type: FileType.video,
  allowedExtensions: ['mp4', 'mov', 'avi'],
);
```

#### 3. Live Workout Integration

```dart
// In LiveWorkoutScreen
MediaPlayerWidget(
  videoUrl: currentExercise.tutorialVideo,
  autoPlay: true,
  showControls: false, // Minimal controls during workout
  onPlaybackComplete: () => moveToNextExercise(),
)
```

## Technical Details

### Dependencies

- `video_player: ^2.8.1` - Core video playback functionality
- Uses Flutter's built-in video player for cross-platform support

### Platform Support

- ✅ iOS (AVFoundation)
- ✅ Android (ExoPlayer)
- ✅ Web (HTML5 video)
- ⚠️ Desktop platforms may need additional configuration

### Video Formats

- Supported formats depend on platform:
  - iOS: MP4, MOV, M4V
  - Android: MP4, WebM, MKV
  - Web: MP4, WebM

### Performance Considerations

- Videos are streamed from network URLs
- No local caching implemented yet
- Consider video compression for mobile data usage
- Placeholder images recommended for better UX

## Next Steps

### Phase 1: Video Upload Support

1. Add video file picker to trainer upload flow
2. Implement video storage in R2
3. Add video URLs to workout plan data model

### Phase 2: Workout Integration

1. Add tutorial videos to exercise database
2. Integrate media player into live workout screen
3. Add video controls to workout flow

### Phase 3: Advanced Features

1. Video caching for offline viewing
2. Video quality selection
3. Picture-in-picture mode
4. Video bookmarks/notes

## Security & Privacy

- Videos are hosted on external CDN (R2)
- No video processing on device
- Network URLs only (no local file access for security)

## Testing

- Demo screen accessible via Alice chat menu
- Sample videos use public test URLs
- Test on both iOS and Android devices
- Verify fullscreen functionality
- Test with different video formats and qualities

## Troubleshooting

### Common Issues

1. **Video not playing**: Check URL format and network connectivity
2. **Controls not showing**: Ensure `showControls=true`
3. **Audio not working**: Check device volume and video audio track
4. **Fullscreen issues**: Verify platform-specific behavior

### Debug Tips

- Use demo screen to test video URLs
- Check console for video player errors
- Test with different video formats
- Verify network connectivity

## Related
