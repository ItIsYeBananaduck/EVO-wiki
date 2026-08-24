---
title: CHAT_PERSISTENCE_IMPLEMENTATION
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CHAT_PERSISTENCE_IMPLEMENTATION.md"]
updated: 2026-07-24
---

# Chat Persistence Implementation Summary

## ✅ Phase 1: Core Persistence - COMPLETE

### Files Created

1. **`chat_message.dart`** - Chat message model
   - Stores individual messages with metadata
   - Supports JSON/JSONL serialization
   - Includes UI blocks and metadata

2. **`chat_session.dart`** - Chat session model
   - Session metadata (ID, domain, timestamps)
   - Context (workoutId, planId)
   - Active/archived status

3. **`chat_history_manager.dart`** - Chat history service
   - Session management (get/create/archive)
   - Message storage (append-only JSONL)
   - Session lifecycle (continue vs. new)
   - Storage: `AliceAssets/chat/{userId}/sessions.json` + `{sessionId}/messages.jsonl`

### Files Modified

1. **`alice_chat_screen.dart`**
   - Added `ChatHistoryManager` initialization
   - Loads chat history on screen open
   - Saves messages when sent/received
   - Gets or creates session (continues existing if appropriate)
   - Builds chat history context for prompts

2. **`alice_brain_service.dart`**
   - Added `chatHistoryContext` to `AliceBrainContext`
   - Enhanced memory brief to include recent chat history
   - Combines RAG memories + recent conversation

## How It Works

### Session Management

**Continue Existing Session If**:

- Same domain (workout/nutrition/general)
- Last active < 24 hours
- Same workout session (if workoutId matches)

**Create New Session If**:

- Domain changes
- Session expired (> 24 hours inactive)
- User explicitly starts new chat

### Message Persistence

1. **On Send**: User message saved immediately
2. **On Receive**: Alice response saved with metadata
3. **On Load**: Last 100 messages loaded when chat opens
4. **Storage**: Append-only JSONL (efficient, recoverable)

### Context Integration

**Memory Brief Now Includes**:

```
[RECENT CONVERSATION]
User: What's my workout plan?
Alice: Here's your plan...
User: Can I modify it?
[/RECENT CONVERSATION]

[MEMORY BRIEF]
• [FACT] User prefers morning workouts
• [TRIED] Interval training worked well
[/MEMORY BRIEF]
```

- **Recent conversation**: Last 10 messages (immediate context)
- **Memory brief**: Relevant facts/preferences (long-term context)

## Storage Structure

```
AliceAssets/chat/
  {userId}/
    sessions.json              # All session metadata
    {sessionId}/
      messages.jsonl          # Chat messages (append-only)
      metadata.json           # Session info (duplicate for quick access)
```

## ✅ Phase 2: Session Management UI - COMPLETE

### Files Created

1. **`chat_session_list_drawer.dart`** - Session list UI
   - Drawer showing all chat sessions
   - Session metadata (domain, message count, last active)
   - Current session highlighting
   - "New Chat" button

### Files Modified

1. **`alice_chat_screen.dart`**
   - Added session management buttons in header
   - `_startNewChat()` - Creates new session and clears messages
   - `_switchToSession()` - Loads existing session messages
   - Session list drawer integration

### Features

- **Session List**: View all active sessions with metadata
- **New Chat**: Start fresh conversation (archives current if has messages)
- **Switch Sessions**: Tap any session to load its messages
- **Visual Indicators**: Current session highlighted with checkmark
- **Domain Icons**: Visual distinction for workout/nutrition/recovery/etc.

## Next Steps (Future Phases)

### Phase 3: Live Activities (iOS)

- [ ] ActivityKit integration
- [ ] Dynamic Island chat widget
- [ ] Lock screen chat access

### Phase 4: Android Bubbles

- [ ] BubbleService implementation
- [ ] Floating chat bubble
- [ ] Notification integration

### Phase 5: Advanced Features

- [ ] Cross-device sync (optional)
- [ ] Message search
- [ ] Export chat history

## Testing

**To Test**:

1. Send a few messages in chat
2. Close chat screen
3. Reopen chat screen
4. Verify messages are restored
5. Verify session continues (same domain, < 24h)

**Expected Behavior**:

- Messages persist across app restarts
- Session continues if same domain and recent
- New session created if domain changes or expired
- Chat history context included in prompts

## Notes

- **SAF Storage**: Currently skipped on Android SAF (would need SharedModelStore append support)
- **File System**: Full support on iOS and Android file system storage
- **Performance**: JSONL append-only is efficient for large histories
- **Cleanup**: Old sessions auto-archive after 7 days (can be configured)

## Related
