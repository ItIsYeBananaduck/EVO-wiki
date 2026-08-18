---
title: LONG_ANSWER_STRATEGY
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/LONG_ANSWER_STRATEGY.md"]
updated: 2026-07-24
---

# Long Answer Strategy

## Overview

When Alice needs to generate answers that exceed available tokens, the system uses a **multi-pass continuation strategy** with a **sliding window** approach to keep prompts within context limits.

## Current Implementation

### 1. **Initial Generation**

- **Token Limit**: 192-256 tokens per pass (domain-dependent)
- **Domain Limits**:
  - `live_workout`: 256 tokens
  - `nutrition`, `recovery`, `planning`: 384 tokens
  - Other domains: 512 tokens
- **Device Scaling**: Lower-end devices get 70-90% of base limits

### 2. **Continuation Strategy**

The system automatically detects incomplete answers and continues generation:

#### **Agentic Tasks** (up to 3-5 continuations)

- Detects incomplete answers using heuristics:
  - Very short answers (< 30 chars)
  - Mid-sentence endings
  - Incomplete phrases ("I'm going to", "I'll", etc.)
- Continues until task completion or max attempts reached

#### **Capability Queries** (up to 5 continuations)

- Forced continuation for admin queries about capabilities
- More lenient completion check (200+ chars, sentence boundary)
- Designed to provide comprehensive capability lists

#### **Chat Mode** (1 continuation)

- Single continuation pass if answer seems incomplete
- More conservative detection (only for very short or clearly incomplete)

### 3. **Sliding Window Continuation**

Each continuation uses a **sliding window** to stay within context limits:

```
Context Window: 512 tokens (simulator) or larger (device)
├── System prompt: ~50 tokens
├── User message: variable
├── Existing answer snippet: variable (last 200-400 chars)
└── Generation headroom: ~100 tokens
```

**Key Optimizations:**

- **Extracts only user message** from original prompt (not full system prompt)
- **Truncates existing answer** to last N chars (200 for chat, 400 for capability queries)
- **Finds sentence boundaries** when truncating to maintain coherence
- **Calculates available space** dynamically based on context window size

### 4. **Graceful Degradation**

If continuation fails (context window full, decode error):

- Returns empty string (stops continuation loop)
- Uses existing answer (even if incomplete)
- Logs warning for debugging
- No crash or error to user

## Example Flow

**User asks**: "List your agentic capabilities"

1. **Initial Generation** (192 tokens)
   - Generates: "As an AI fitness assistant, my agentic capabilities include: Providing personalized workout plans..."
   - Detects: Answer incomplete (capability query, needs continuation)

2. **Continuation 1** (256 tokens)
   - Prompt: User message + last 400 chars of answer
   - Generates: "...analyzing form, tracking progress, and adapting plans based on your feedback..."
   - Total: ~686 chars

3. **Continuation 2** (256 tokens)
   - Prompt: User message + last 400 chars of combined answer
   - Generates: "...I can also help with nutrition planning, recovery recommendations..."
   - Total: ~1544 chars

4. **Stop Condition**
   - Either: Answer seems complete (200+ chars, sentence boundary, no "...")
   - Or: Continuation returns empty (context window full)
   - Or: Max attempts reached (5 for capability queries)

## Seamless Continuation (No Mid-Sentence Cutoffs)

### Sentence Boundary Detection

The system uses multiple strategies to prevent mid-sentence cutoffs:

1. **Generation-Time Detection**:
   - `sentenceBoundaryBuffer`: Allows 10 tokens over limit to find sentence end
   - Checks for sentence boundaries (`.`, `!`, `?` + whitespace) when approaching token limit
   - Stops gracefully at sentence boundaries instead of hard cutoff

2. **Continuation Truncation**:
   - Uses `findLastSentenceBoundary()` to find proper sentence ends
   - Falls back to word boundaries (spaces/newlines) if no sentence boundary
   - Only cuts mid-word as last resort (with warning)
   - Ensures continuation starts naturally, not mid-sentence

3. **Continuation Instructions**:
   - Explicitly instructs model: "Continue seamlessly from where it left off"
   - Emphasizes: "Make sure the continuation flows naturally as if it's all one continuous answer"
   - Prevents repetition and ensures smooth flow

### Example Flow

```
Initial: "I can help with workout planning, form analysis..."
         ↑ (hits token limit, but finds sentence boundary)

Continuation 1: "...tracking progress, and adapting plans based on your feedback."
                ↑ (ends at sentence boundary)

Continuation 2: "I can also help with nutrition planning..."
                ↑ (seamless continuation, no repetition)
```

## Conversation History Management

**Current State**: Stateless - each request is independent

- No conversation history is maintained
- Only current user message + optional `memoryBrief` (RAG context)
- No summarization of previous messages needed

**Why No Summarization?**

- Each request is self-contained
- `memoryBrief` provides relevant context from RAG (not conversation history)
- Context window is used for current prompt + continuation, not history

**If Conversation History Needed**:

- Would need to implement message history tracking
- Would need summarization when history exceeds context window
- Would need to decide: keep recent messages vs. summarize older ones

## Inference Performance (Lightweight Design)

### Continuation is Conservative

The system is designed to keep inference lightweight:

1. **Rarely Triggers**:
   - **Chat mode**: Only 1 continuation, and only if answer is very short (< 50 chars) OR clearly incomplete (mid-sentence, incomplete phrases)
   - **Agentic mode**: Only if answer indicates task incomplete (heuristics are strict)
   - **Capability queries**: Only for admin queries about capabilities (rare)

2. **Small Token Budgets**:
   - **Chat continuation**: `min(128, domainMaxTokens / 2)` - very small (64-128 tokens)
   - **Agentic continuation**: `min(256, domainMaxTokens)` - moderate (192-256 tokens)
   - **Capability continuation**: `min(256, domainMaxTokens)` - moderate (192-256 tokens)

3. **Early Stopping**:
   - Stops immediately if continuation returns empty (context window full)
   - Stops if continuation doesn't add enough content (> 10 chars)
   - Stops if answer seems complete (sentence boundary, reasonable length)

4. **Optimized Prompts**:
   - Sliding window: Only includes last 200-2000 chars (not full answer)
   - Extracts only user message (not full system prompt with capability map)
   - Compact continuation prompts minimize token usage

### Performance Impact

- **Most queries**: No continuation (0 extra inference passes)
- **Short incomplete answers**: 1 small continuation pass (~64-128 tokens)
- **Complex agentic tasks**: 1-3 moderate continuation passes (~256 tokens each)
- **Capability queries**: 1-5 moderate continuation passes (admin only, rare)

**Typical overhead**: < 5% of queries trigger continuation, and those use small token budgets.

## Limitations

1. **Context Window**: 512 tokens on simulator, up to 2048 on high-end devices
2. **Token Estimation**: Uses rough 4 chars/token estimate (not exact)
3. **No Streaming**: Continuation happens before returning to user (not real-time)
4. **Fixed Limits**: Max 5 continuations even if answer still incomplete
5. **No Conversation History**: Each request is stateless (no previous messages)

## Future Improvements

1. **Dynamic Context Window**: Use larger context on devices that support it
2. **Exact Token Counting**: Use actual tokenizer to count tokens precisely
3. **Streaming Continuation**: Stream partial answer, then continue in background
4. **Adaptive Limits**: Increase max continuations for very long queries
5. **Chunking Strategy**: For extremely long answers, break into logical sections

## Code Locations

- **Token Limits**: `getMaxTokensForDomain()` (line ~3179)
- **Continuation Logic**: `generateVoiceAnswerContinuation()` (line ~5291)
- **Incompleteness Detection**: `isAnswerIncomplete()` (line ~5234)
- **Continuation Orchestration**: Lines ~2410-2537 in `_processGeneration()`

## Related

^[{src_rel}]
