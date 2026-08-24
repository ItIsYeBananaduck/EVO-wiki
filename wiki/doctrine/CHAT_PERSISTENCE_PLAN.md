---
title: CHAT_PERSISTENCE_PLAN
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/CHAT_PERSISTENCE_PLAN.md
updated: 2026-07-24
---

# Chat Persistence & Live Activities Integration Plan

## Overview

Enable chat to persist across app lifecycle and remain accessible during Live Activities (iOS) and Android equivalent, allowing users to continue conversations even when the app is backgrounded or during active workouts.

## Current State

### Chat Session Management

- **Session ID**: Generated per chat instance (`_sessionId` in `alice_chat_screen.dart`)
- **Messages**: Stored in memory (`_messages` list) - **NOT persisted**
- **Session Lifecycle**: New session created on each chat screen open
- **History**: Lost when user exits chat or app closes

### Existing Memory System (RAG)

- **ConversationMemoryManager**: Stores semantic/episodic/procedural memories (RAG-style)
- **What it stores**: Important events/facts extracted from conversations, NOT full chat messages
- **Purpose**: RAG retrieval - finds relevant memories based on query keywords
- **Storage**: `AliceAssets/memory/{appId}/{userId}/memories.jsonl`
- **Usage**: Builds `MemoryBrief` injected into prompts for context
- **Limitation**: Only stores "events" (preferences, decisions, facts) - not full conversation flow

**Key Difference**:

- **Memory System**: Stores extracted facts/events for RAG retrieval (what happened, what was tried, facts learned)
- **Chat History**: Would store full message-by-message conversation (for display and continuity)

**We can leverage both**:

- Use memory system for long-term context (facts, preferences)
- Use chat history for recent conversation flow and session continuity

## Requirements

1. **Persist Chat History**: Save full conversation when user exits
2. **Session Management**:
   - When to start new session vs. continue existing
   - When to close/archive session
   - How to reload existing session
3. **Live Activities Integration**:
   - Keep chat accessible during workouts (iOS Live Activities)
   - Android equivalent (persistent notifications or bubbles)
4. **Cross-Session Context**: Load relevant history when starting new chat

## Proposed Solution

### 1. Chat History Storage

**New Service**: `ChatHistoryManager`

```dart
class ChatHistoryManager {
  // Store chat sessions
  Future<void> saveSession(ChatSession session);
  Future<ChatSession?> loadSession(String sessionId);
  Future<List<ChatSession>> listSessions({int limit = 50});
  Future<void> archiveSession(String sessionId);

  // Store messages within session
  Future<void> appendMessage(String sessionId, ChatMessage message);
  Future<List<ChatMessage>> loadMessages(String sessionId);
}
```

**Storage Structure**:

```
AliceAssets/chat/
  {userId}/
    sessions.json          # Session metadata
    {sessionId}/
      messages.jsonl       # Chat messages (append-only)
      metadata.json        # Session info (domain, created, lastActive)
```

**Session Schema**:

```json
{
  "sessionId": "uuid",
  "userId": "user-id",
  "domain": "workout|nutrition|general",
  "createdAt": "ISO8601",
  "lastActiveAt": "ISO8601",
  "messageCount": 42,
  "isActive": true,
  "isArchived": false,
  "context": {
    "workoutId": "optional",
    "planId": "optional"
  }
}
```

**Message Schema**:

```json
{
  "id": "uuid",
  "sessionId": "uuid",
  "text": "message content",
  "isUser": true,
  "timestamp": "ISO8601",
  "uiBlocks": [...],  // Optional UI blocks
  "metadata": {
    "tokens": 150,
    "responseTime": 1200
  }
}
```

### 2. Session Lifecycle Management

**When to Start New Session**:

- User explicitly starts new chat (button)
- Domain changes (workout → nutrition)
- Session expired (inactive > 24 hours)
- User clears chat history

**When to Continue Existing Session**:

- User reopens chat within same domain
- Same workout session (if workoutId matches)
- Last active < 24 hours ago

**When to Archive Session**:

- User explicitly ends session
- Session inactive > 7 days
- App uninstall/reinstall (optional)

**Session Selection Logic**:

```dart
Future<String> getOrCreateSession({
  required String userId,
  required AliceCoachingDomain domain,
  String? workoutId,
  String? planId,
}) async {
  // 1. Check for active session in same domain
  final activeSession = await findActiveSession(
    userId: userId,
    domain: domain,
    workoutId: workoutId,
  );

  if (activeSession != null &&
      activeSession.lastActiveAt.isAfter(DateTime.now().subtract(Duration(hours: 24)))) {
    return activeSession.sessionId;  // Continue existing
  }

  // 2. Create new session
  return await createNewSession(
    userId: userId,
    domain: domain,
    workoutId: workoutId,
    planId: planId,
  );
}
```

### 3. Chat History Loading

**On Chat Screen Open**:

```dart
@override
void initState() {
  super.initState();
  _loadChatHistory();
}

Future<void> _loadChatHistory() async {
  // 1. Get or create session
  final sessionId = await chatHistoryManager.getOrCreateSession(
    userId: widget.user.id,
    domain: widget.domain,
    workoutId: currentWorkoutId,
  );

  // 2. Load messages
  final messages = await chatHistoryManager.loadMessages(sessionId);

  // 3. Restore UI state
  setState(() {
    _sessionId = sessionId;
    _messages = messages.map((m) => _ChatMessage(
      text: m.text,
      isUser: m.isUser,
      uiBlocks: m.uiBlocks,
    )).toList();
  });

  // 4. Scroll to bottom
  WidgetsBinding.instance.addPostFrameCallback((_) {
    _scrollToBottom();
  });
}
```

**Auto-Save on Message**:

```dart
Future<void> _sendMessage(String text) async {
  // ... existing send logic ...

  // Save user message
  await chatHistoryManager.appendMessage(_sessionId, ChatMessage(
    text: text,
    isUser: true,
    timestamp: DateTime.now(),
  ));

  // Save Alice response when received
  // (in response handler)
  await chatHistoryManager.appendMessage(_sessionId, ChatMessage(
    text: response,
    isUser: false,
    timestamp: DateTime.now(),
    uiBlocks: responseUiBlocks,
  ));

  // Update session lastActiveAt
  await chatHistoryManager.updateSessionActivity(_sessionId);
}
```

### 4. Live Activities Integration (iOS)

**iOS Live Activities**:

- Use `ActivityKit` framework
- Show chat interface in Dynamic Island / Lock Screen
- Keep chat accessible during workouts

**Implementation**:

```swift
// iOS: LiveActivityChatWidget.swift
import ActivityKit
import WidgetKit

struct ChatLiveActivity: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: ChatAttributes.self) { context in
            // Chat UI in Live Activity
            ChatView(sessionId: context.attributes.sessionId)
        } dynamicIsland: { context in
            DynamicIsland {
                // Compact view in Dynamic Island
                expandedLeading: {
                    // Alice avatar
                }
                expandedTrailing: {
                    // Last message preview
                }
                compactLeading: {
                    // Alice icon
                }
                compactTrailing: {
                    // Unread count
                }
            }
        }
    }
}
```

**Start Live Activity**:

```dart
// Flutter: Start Live Activity when workout starts
if (Platform.isIOS && isActiveWorkout) {
  await LiveActivityManager.startChatActivity(
    sessionId: _sessionId,
    domain: widget.domain,
  );
}
```

**Update Live Activity**:

```dart
// When new message arrives
await LiveActivityManager.updateChatActivity(
  sessionId: _sessionId,
  lastMessage: message.text,
  unreadCount: unreadCount,
);
```

**End Live Activity**:

```dart
// When workout ends or user closes chat
await LiveActivityManager.endChatActivity(sessionId: _sessionId);
```

### 5. Android Equivalent

**Android Options**:

1. **Bubbles API** (Android 11+): Floating chat bubble
2. **Persistent Notification**: Expandable notification with chat UI
3. **Picture-in-Picture**: Chat overlay during workouts

**Recommended**: Bubbles API for best UX

**Implementation**:

```kotlin
// Android: ChatBubbleService.kt
class ChatBubbleService : FloatingService() {
    override fun onCreate() {
        super.onCreate()
        // Create bubble with chat UI
    }

    fun showBubble(sessionId: String) {
        val bubbleIntent = Intent(this, ChatActivity::class.java)
            .putExtra("sessionId", sessionId)

        val bubbleMetadata = BubbleMetadata.Builder()
            .setIntent(bubbleIntent)
            .setIcon(Icon.createWithResource(this, R.drawable.alice_icon))
            .build()

        val notification = Notification.Builder(this, CHANNEL_ID)
            .setBubbleMetadata(bubbleMetadata)
            .build()

        startForeground(1, notification)
    }
}
```

### 6. Session Context Loading (Leveraging Existing RAG)

**Combine Chat History + Existing Memory System**:

```dart
Future<String> buildMemoryBriefWithHistory({
  required String query,
  required String sessionId,
  String? domain,
}) async {
  // 1. Get recent messages from current session (NEW - chat history)
  final recentMessages = await chatHistoryManager.loadMessages(
    sessionId,
    limit: 10,  // Last 10 messages for immediate context
  );

  // 2. Get relevant memories (EXISTING - RAG system)
  // This already exists and works well for long-term context
  final memories = await conversationMemoryManager.buildMemoryBrief(
    query: query,
    mode: domain,
  );

  // 3. Combine into context (recent chat + long-term memories)
  final buffer = StringBuffer();

  // Recent conversation provides immediate context
  if (recentMessages.isNotEmpty) {
    buffer.writeln('[RECENT CONVERSATION]');
    for (final msg in recentMessages) {
      buffer.writeln('${msg.isUser ? "User" : "Alice"}: ${msg.text}');
    }
    buffer.writeln('[/RECENT CONVERSATION]');
  }

  // Long-term memories provide facts/preferences/learned patterns
  if (memories.isNotEmpty) {
    buffer.writeln(memories);  // Already formatted as [MEMORY BRIEF]...[/MEMORY BRIEF]
  }

  return buffer.toString();
}
```

**Why Both?**:

- **Chat History**: Provides conversation flow and immediate context (last 10 messages)
- **Memory System (RAG)**: Provides long-term facts, preferences, and learned patterns
- **Together**: Alice has both recent conversation context AND long-term knowledge

## Implementation Steps

### Phase 1: Core Persistence

1. [ ] Create `ChatHistoryManager` service
2. [ ] Implement session storage (sessions.json)
3. [ ] Implement message storage (messages.jsonl)
4. [ ] Update `AliceChatScreen` to save/load messages
5. [ ] Test persistence across app restarts

### Phase 2: Session Management

1. [ ] Implement session lifecycle logic
2. [ ] Add session selection (new vs. continue)
3. [ ] Add session archiving
4. [ ] Add session list UI (optional)
5. [ ] Test session continuity

### Phase 3: Live Activities (iOS)

1. [ ] Create Live Activity widget
2. [ ] Implement ActivityKit integration
3. [ ] Add Dynamic Island support
4. [ ] Test during workouts
5. [ ] Handle activity updates

### Phase 4: Android Bubbles

1. [ ] Create BubbleService
2. [ ] Implement bubble UI
3. [ ] Add notification channel
4. [ ] Test bubble functionality
5. [ ] Handle bubble lifecycle

### Phase 5: Context Integration

1. [ ] Load recent messages into context
2. [ ] Combine with memory brief
3. [ ] Update prompt building
4. [ ] Test context continuity
5. [ ] Optimize context size

## File Locations

**New Files**:

- `flutter_app/lib/features/alice/domain/chat_history_manager.dart`
- `flutter_app/lib/features/alice/domain/chat_session.dart`
- `flutter_app/lib/features/alice/domain/chat_message.dart`
- `flutter_app/ios/Runner/LiveActivityChatWidget.swift` (iOS)
- `flutter_app/android/app/src/main/kotlin/.../ChatBubbleService.kt` (Android)

**Modified Files**:

- `flutter_app/lib/features/alice/presentation/alice_chat_screen.dart`
- `flutter_app/lib/features/alice/domain/alice_brain_service.dart`
- `flutter_app/lib/features/alice/domain/conversation_memory_manager.dart`

## Storage Considerations

**Size Limits**:

- Max messages per session: 1000 (then archive)
- Max sessions per user: 100 (then archive oldest)
- Message size limit: 10KB per message
- Total storage per user: ~50MB max

**Cleanup Strategy**:

- Archive sessions inactive > 7 days
- Delete archived sessions > 30 days old
- Compress old messages (optional)

## Testing Checklist

- [ ] Chat persists across app restarts
- [ ] Session continues when reopening chat
- [ ] New session created when appropriate
- [ ] Live Activity shows chat during workout
- [ ] Android bubble works correctly
- [ ] Context loads from history
- [ ] Performance acceptable with large history
- [ ] Storage cleanup works
- [ ] Cross-device sync (if needed)

## Related

^[source-materials/mirrors/doctrine/CHAT_PERSISTENCE_PLAN.md]
