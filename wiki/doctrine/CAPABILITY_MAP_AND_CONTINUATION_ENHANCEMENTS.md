---
title: CAPABILITY_MAP_AND_CONTINUATION_ENHANCEMENTS
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/CAPABILITY_MAP_AND_CONTINUATION_ENHANCEMENTS.md
updated: 2026-07-24
---

# Capability Map & Continuation Enhancements

## Two Key Improvements Needed

1. **Capability Map for "I Don't Know" Scenarios**: Alice should use capability map when she doesn't know an answer (especially for admin)
2. **Continuation Until Task Complete**: Alice should continue inference until tasks are complete, even if token limits are hit

---

## Issue 1: Capability Map for Unknown Answers

### Current Behavior

- Capability map only loaded for specific scenarios (action requests, admin technical questions)
- If Alice doesn't know an answer to a general question, map isn't available
- Admin might ask about capabilities but map not loaded

### Solution: Always Include Map for Admins

**Updated Logic:**

- **Admins**: Always get capability map (they may ask about capabilities or need action details)
- **Users**: Map only when action/query detected (keeps inference lite)

**Implementation:**

```kotlin
// Android
if (isAdmin) {
    return true  // Always include for admin
}
```

```swift
// iOS
if isAdmin {
    return true  // Always include for admin
}
```

**Benefit**: Admins can always ask about capabilities and get detailed answers

---

## Issue 2: Continuation Until Task Complete

### Current Behavior (iOS)

- Continuation logic exists for agentic tasks (up to 3 attempts)
- Checks if answer is incomplete
- Continues generation until task complete

### Current Behavior (Android)

- **Missing**: No continuation logic
- Needs to be implemented

### Solution: Implement Continuation on Android + Enhance iOS

**Requirements:**

1. Detect incomplete answers (especially for agentic tasks)
2. Continue inference until task is complete
3. Handle token limits gracefully
4. Support multi-turn continuation

---

## Implementation Plan

### Phase 1: Admin Always Gets Capability Map ✅

**Status**: Already implemented

- Admins always get capability map
- Users get map only when needed

### Phase 2: Enhanced Continuation Logic

#### Android Implementation Needed

```kotlin
// After generating response, check if incomplete
val isIncomplete = isAnswerIncomplete(parsed.answer, parsed.actions)
if (isIncomplete && hasAgenticActions) {
    // Continue until complete
    var currentAnswer = parsed.answer
    var attempts = 0
    val maxAttempts = 3

    while (attempts < maxAttempts && isAnswerIncomplete(currentAnswer, parsed.actions)) {
        attempts++
        val continuation = generateContinuation(
            originalPrompt = systemPrompt,
            existingAnswer = currentAnswer,
            maxTokens = 256
        )
        if (continuation.isNotEmpty()) {
            currentAnswer += " " + continuation
        } else {
            break
        }
    }

    parsed.answer = currentAnswer
}
```

#### iOS Enhancement

Current continuation logic is good, but we should:

1. Increase max attempts for complex tasks (admin mode)
2. Better detection of incomplete answers
3. Support for very long tasks

---

## Detection Logic

### Incomplete Answer Detection

**Signals that answer is incomplete:**

1. Very short answer (< 30 chars for agentic tasks)
2. Ends with incomplete sentence (comma, dash, "and")
3. Action present but answer doesn't explain it
4. Answer doesn't address the user's request

**Implementation:**

```kotlin
private fun isAnswerIncomplete(answer: String, hasAgenticActions: Boolean): Boolean {
    val trimmed = answer.trim()

    // Very short answers are likely incomplete
    if (trimmed.length < 30 && hasAgenticActions) {
        return true
    }

    // Ends with incomplete markers
    val lastChar = trimmed.lastOrNull()
    if (lastChar == ',' || lastChar == '—' || lastChar == '-') {
        return true
    }

    // Doesn't end with punctuation (might be cut off)
    if (!trimmed.endsWith('.') && !trimmed.endsWith('!') && !trimmed.endsWith('?')) {
        if (trimmed.length < 60) {  // Short answers without punctuation are suspicious
            return true
        }
    }

    return false
}
```

---

## Token Limit Handling

### Current Limits

- **iOS**: Domain-based limits (live_workout=128, others=256-384)
- **Android**: Needs to be implemented

### Strategy for Long Tasks

1. **First Pass**: Generate policy + actions (short, ~128 tokens)
2. **Second Pass**: Generate answer (up to domain limit)
3. **Continuation**: If incomplete, continue (up to 3 attempts, 256 tokens each)
4. **Total Possible**: ~128 + 384 + (3 × 256) = ~1280 tokens for complex tasks

### Admin Mode: Higher Limits

For admin queries, we should allow longer responses:

- **Admin**: Up to 512 tokens per pass
- **Continuation**: Up to 5 attempts for admin
- **Total Possible**: ~512 + 512 + (5 × 512) = ~3584 tokens for admin tasks

---

## Updated System Prompt Instructions

When capability map is included, add instructions about continuation:

```
CAPABILITY_MAP USAGE:
- The CAPABILITY_MAP below shows available actions and how to use them
- When user requests an action, find it in the map and follow the 'howToUse' steps
- Check 'whenToUse' to confirm it's the right action for this situation
- Verify 'access' rules match your INTERNAL_CONFIG permissions
- Use 'payload' structure to format the action correctly
- If your answer is incomplete, the system will continue generation until the task is complete
- For complex tasks, provide complete explanations even if it requires multiple passes
```

---

## Testing Scenarios

### Scenario 1: Admin Asks About Capabilities

**User (Admin)**: "What can you do?"

- ✅ Capability map loaded (admin always gets it)
- ✅ Alice can reference map to answer
- ✅ Continuation available if answer is long

### Scenario 2: Complex Task Exceeds Tokens

**User**: "Create a detailed 6-day push/pull/legs plan with specific exercises, sets, reps, and progression strategy"

- ✅ Capability map loaded (action request detected)
- ✅ First pass: Generate policy + actions
- ✅ Second pass: Generate answer (may hit token limit)
- ✅ Continuation: Continue until complete explanation provided

### Scenario 3: General Question (No Map Needed)

**User**: "How are you today?"

- ✅ No capability map (keeps inference lite)
- ✅ Simple answer sufficient
- ✅ No continuation needed

---

## Files to Modify

### Android

- `LlamaPlugin.kt`
  - ✅ Already: Admin always gets map
  - ⚠️ **Need**: Add continuation logic
  - ⚠️ **Need**: Add incomplete answer detection
  - ⚠️ **Need**: Add token limit handling

### iOS

- `LlamaEngine.swift`
  - ✅ Already: Continuation logic exists
  - ✅ Already: Admin always gets map (after our update)
  - ⚠️ **Enhance**: Better incomplete detection
  - ⚠️ **Enhance**: Higher limits for admin

---

## Next Steps

1. ✅ **Done**: Admin always gets capability map
2. ⚠️ **Todo**: Implement continuation logic on Android
3. ⚠️ **Todo**: Enhance continuation detection
4. ⚠️ **Todo**: Add admin-specific token limits
5. ⚠️ **Todo**: Test with complex tasks

---

**Status**: Phase 1 complete, Phase 2 needs implementation

## Related

^[source-materials/mirrors/doctrine/CAPABILITY_MAP_AND_CONTINUATION_ENHANCEMENTS.md]
