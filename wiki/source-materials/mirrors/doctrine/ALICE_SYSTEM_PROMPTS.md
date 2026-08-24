---
title: ALICE_SYSTEM_PROMPTS
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/ALICE_SYSTEM_PROMPTS.md"]
updated: 2026-07-24
---

# Alice System Prompts Reference

> Complete documentation of all system prompts used by Alice AI fitness coach.

## Architecture Overview

Alice uses a **layered prompt system**:

```
┌─────────────────────────────────────────┐
│           Role Overlay                  │  ← admin vs user permissions
├─────────────────────────────────────────┤
│         Autonomy Overlay                │  ← guided/collaborative/autonomous
├─────────────────────────────────────────┤
│          Domain Overlay                 │  ← workout/nutrition/recovery/etc
├─────────────────────────────────────────┤
│         Core System Prompt              │  ← base personality & style
└─────────────────────────────────────────┘
```

---

## Pipeline Sequence (Current Flow)

The full inference pipeline from user message to displayed response:

```
┌───────────────────────────────────────────────────────────────────────────┐
│                          FLUTTER LAYER                                     │
├───────────────────────────────────────────────────────────────────────────┤
│  1. AliceBrainRequest received                                            │
│     └─ Contains: userMessage, context.domain, context.user                │
│                                                                           │
│  2. Resolve Autonomy Policy (async)                                       │
│     └─ AliceAutonomyService.resolveAutonomyPolicy()                       │
│     └─ Returns: EffectiveAutonomyPolicy { mode, source, trainerIdentifier }│
│                                                                           │
│  3. Load Guardrails (async)                                               │
│     └─ AliceGuardrailService.loadGuardrails()                             │
│     └─ Returns: AliceGuardrailBundle { versionLabel, rules }              │
│                                                                           │
│  4. Build Adapter Stack                                                   │
│     └─ _buildAdapterStack(user, autonomy)                                 │
│     └─ Returns: { type: 'client'|'trainer', adapters: [...] }             │
│                                                                           │
│  5. Platform Channel Call ───────────────────────────────────────────────►│
│     └─ MethodChannel('evo/alice_brain').invokeMethod('generate', {...})   │
└───────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                          iOS NATIVE LAYER (LlamaEngine.swift)             │
├───────────────────────────────────────────────────────────────────────────┤
│  6. Queue Management                                                      │
│     └─ generationQueue.async { ... }                                      │
│     └─ If already generating → queue request (overwrites pending)         │
│     └─ state = .generating                                                │
│                                                                           │
│  7. Intent Gate (CURRENTLY DISABLED)                                      │
│     └─ handleIntentGate() would short-circuit for greetings               │
│     └─ Code present but commented out                                     │
│                                                                           │
│  8. Model Load Check                                                      │
│     └─ If !_isLoaded → loadModel() → fallback if fails                    │
│                                                                           │
│  9. Build System Prompt ◄──────── PROMPT ASSEMBLY HAPPENS HERE            │
│     └─ buildSystemPrompt(domain, autonomyMode, role, isActiveWorkout)     │
│     │                                                                     │
│     └─► buildCoreSystem()              → base personality (always)        │
│     └─► buildDomainOverlay()           → context-specific (workout/etc)   │
│     └─► buildAutonomyOverlay()         → coaching style (guided/collab)   │
│     └─► buildRoleOverlay()             → permissions (admin/user)         │
│     └─► buildResponsePolicyOverlay()   → ✅ NEW: structured output format │
│     │                                                                     │
│     └─ Layers joined with "\n\n"                                          │
│                                                                           │
│ 10. Format as Mistral Instruct                                            │
│     └─ "<s>[INST] <<SYS>>\n{systemPrompt}\n<</SYS>>\n\n{userMessage}\n[/INST]"│
│                                                                           │
│ 11. Tokenize                                                              │
│     └─ llama_tokenize(vocab, fullPrompt, ...)                             │
│                                                                           │
│ 12. Clear KV Cache                                                        │
│     └─ llama_memory_clear(memory, true) → fresh generation each call      │
│                                                                           │
│ 13. Decode Prompt Tokens                                                  │
│     └─ llama_batch_init() → fill batch → llama_decode()                   │
│                                                                           │
│ 14. Sampling Loop (per-domain token cap)                                  │
│     └─ Max tokens: live_workout=128, nutrition/recovery=256, general=384  │
│     └─ Create sampler chain: top_k(40) → top_p(0.95) → temp(0.8) → dist   │
│     └─ Loop: sample → accept → detokenize → append text → decode next     │
│     └─ Stop on: EOS token, "</s>", or 500+ chars                          │
│                                                                           │
│ 15. Parse Structured Response ◄────── ✅ NEW: POLICY + CHUNK PARSING      │
│     └─ parseStructuredResponse(rawText)                                   │
│     └─ Extract <policy>{...}</policy> → parse JSON                        │
│     └─ Extract <answer>...</answer> → check for <chunk> tags              │
│     └─ If parsing fails → use default policy + fallback text              │
│                                                                           │
│ 16. Build AliceResponse                                                   │
│     └─ text = first chunk (or full answer if no chunking)                 │
│     └─ uiSpec = { policy, hasChunks, chunks[] }  ◄── NOW POPULATED!       │
│                                                                           │
│ 17. Return via completion handler ◄──────────────────────────────────────►│
└───────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                          FLUTTER POST-PROCESSING                          │
├───────────────────────────────────────────────────────────────────────────┤
│ 18. Parse Response                                                        │
│     └─ rawText = result['text']                                           │
│     └─ policy = AliceResponsePolicy.fromJson(result['uiSpec']['policy'])  │
│     └─ chunks = result['uiSpec']['chunks']                                │
│     └─ hasChunks = result['uiSpec']['hasChunks']                          │
│                                                                           │
│ 19. Apply Guardrails (FitnessTextGuardrail.enforce)                       │
│     └─ Check _disallowedPatterns (mental health, therapy, etc.)           │
│     └─ If matched → replace with fallback fitness message                 │
│     └─ Apply to text AND each chunk                                       │
│     └─ Admin users skip domain enforcement (safety still applies)         │
│                                                                           │
│ 20. Return AliceBrainResponse                                             │
│     └─ { text, autonomyMode, guardrailVersion, policy, chunks, hasChunks }│
└───────────────────────────────────────────────────────────────────────────┘
```

### Key Observations

| Stage           | Location                     | Notes                                                          |
| --------------- | ---------------------------- | -------------------------------------------------------------- |
| Prompt Assembly | LlamaEngine.swift:993        | `buildSystemPrompt()` - 5 layers including response policy     |
| Response Policy | LlamaEngine.swift:1018       | `buildResponsePolicyOverlay()` - ✅ NEW                        |
| Template Format | LlamaEngine.swift:652        | Mistral `[INST]...[/INST]` format hardcoded                    |
| Intent Gate     | LlamaEngine.swift:565        | **DISABLED** - code commented out                              |
| Response Parser | LlamaEngine.swift:1050       | `parseStructuredResponse()` - ✅ NEW: extracts policy + chunks |
| Token Caps      | LlamaEngine.swift:1140       | `getMaxTokensForDomain()` - ✅ NEW: per-domain limits          |
| uiSpec          | LlamaEngine.swift:915        | **NOW POPULATED** with policy, hasChunks, chunks[]             |
| Post-Guardrail  | alice_brain_service.dart:355 | Runs **after** native inference, applied to text + chunks      |

---

## Adding a Pre-Response JSON Stage

**Feasibility: ✅ IMPLEMENTED** - See "Response Policy Contract" section below.

### Option A: Native-Side JSON Prefix (IMPLEMENTED)

Add a JSON reasoning block **before** the natural language response, parsed on the native side.

**Injection Point:** Between steps 9-10 in `LlamaEngine.swift`

```swift
// In buildSystemPrompt() or as new method buildAgenticPrefix()

// Modify the prompt to request JSON-first output:
let agenticInstruction = """
Before responding, output a JSON block wrapped in <think>...</think> tags:
{
  "intent": "log_workout|ask_clarification|provide_guidance|schedule|delegate",
  "confidence": 0.0-1.0,
  "entities": { "exercise": "...", "sets": N, "reps": N },
  "action_required": true|false,
  "delegate_to": null|"trainer"|"server"
}
Then provide your natural language response.
"""

// New fullPrompt format:
let fullPrompt = """
<s>[INST] <<SYS>>
\(sys)

\(agenticInstruction)
<</SYS>>

\(userMessage)
[/INST]
"""
```

**Parsing (add after step 14):**

```swift
// After generation, before AliceResponse creation:
func parseAgenticResponse(_ raw: String) -> (json: [String: Any]?, text: String) {
    if let thinkStart = raw.range(of: "<think>"),
       let thinkEnd = raw.range(of: "</think>") {
        let jsonStr = String(raw[thinkStart.upperBound..<thinkEnd.lowerBound])c
        let textPart = String(raw[thinkEnd.upperBound...]).trimmingCharacters(in: .whitespacesAndNewlines)
        if let data = jsonStr.data(using: .utf8),
           let json = try? JSONSerialization.jsonObject(with: data) as? [String: Any] {
            return (json, textPart)
        }
    }
    return (nil, raw)
}

// Use in response building:
let (agenticJson, cleanedText) = parseAgenticResponse(generatedText)
let response = AliceResponse(
    text: cleanedText,
    uiSpec: agenticJson ?? [:]  // ← Now populated!
)
```

### Option B: Flutter-Side Parsing

Parse JSON from raw text in Dart after receiving from native.

**Injection Point:** Between steps 17-18 in `alice_brain_service.dart`

```dart
// After getting rawText from channel:
final (Map<String, dynamic>? agentic, String cleanText) = _parseAgenticBlock(rawText);

// Apply guardrails to cleanText only
final FitnessTextGuardrailResult gated = FitnessTextGuardrail.enforce(
  cleanText,  // ← not rawText
  enforceDomain: !isAdmin,
);

// Include agentic metadata in response
return AliceBrainResponse(
  text: gated.sanitized,
  agenticIntent: agentic,  // ← new field
  ...
);
```

### Option C: Two-Pass Generation

Run inference twice: once for JSON intent, once for response.

**Pros:** Clean separation, can branch on intent before generating response
**Cons:** 2x latency, 2x compute

### Recommendation

**Option A (Native JSON Prefix)** is best because:

1. Single inference pass
2. Parsing happens close to generation (fewer layers to debug)
3. `uiSpec` field already exists and is unused
4. 3B model can reliably produce JSON with `<think>` wrapper

### Required Changes for Option A

| File                                                                                            | Change                                                       |
| ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| [LlamaEngine.swift](flutter_app/ios/Runner/LlamaEngine.swift#L976)                              | Add agentic instruction to `buildSystemPrompt()`             |
| [LlamaEngine.swift](flutter_app/ios/Runner/LlamaEngine.swift#L890)                              | Add `parseAgenticResponse()` before `AliceResponse` creation |
| [alice_brain_service.dart](flutter_app/lib/features/alice/domain/alice_brain_service.dart#L255) | Consume `uiSpec['intent']` for action dispatch               |
| `AliceBrainResponse`                                                                            | Add `agenticIntent` field                                    |

---

## Response Policy Contract (IMPLEMENTED)

> Added: January 2026. Status: ✅ Active

Alice now uses a **structured response format** that forces the model to:

1. Decide verbosity before responding
2. Think/plan before speaking (without leaking reasoning)
3. Support smart chunking for progressive UI rendering

### How It Works

The model outputs two structured blocks:

```
<policy>
{"verbosity":"short","mode":"action","needsThinking":false,"chunking":"none","maxTokens":128,"firstChunkTokens":48}
</policy>
<answer>
Your response here (or with <chunk> tags if chunking enabled)
</answer>
```

### Policy JSON Schema

| Field              | Type    | Values                                               | Description                                                             |
| ------------------ | ------- | ---------------------------------------------------- | ----------------------------------------------------------------------- |
| `verbosity`        | string  | `"short"` \| `"medium"` \| `"long"`                  | Response length: short=1-2 sentences, medium=1 paragraph, long=detailed |
| `mode`             | string  | `"action"` \| `"explanation"` \| `"mixed"`           | Response type: doing something vs explaining why/how                    |
| `needsThinking`    | boolean | `true` \| `false`                                    | Whether complex reasoning was needed (never revealed to user)           |
| `chunking`         | string  | `"none"` \| `"summary_then_details"` \| `"stepwise"` | How response should be chunked for UI                                   |
| `maxTokens`        | integer | 64-384                                               | Recommended token limit for response                                    |
| `firstChunkTokens` | integer | 32-96                                                | Target size for first chunk (if chunking enabled)                       |

### Chunking Format

When `chunking != "none"`, the answer uses indexed chunks:

```xml
<answer>
<chunk index="1">Immediately useful summary or direct answer.</chunk>
<chunk index="2">Additional details or elaboration.</chunk>
<chunk index="3">Further information if needed.</chunk>
</answer>
```

- **Chunk 1**: Always immediately actionable/useful
- **Chunk 2+**: Additional details, shown progressively

### Per-Domain Token Caps

Hard limits enforced at generation time:

| Domain                              | Max Tokens | Rationale                                |
| ----------------------------------- | ---------- | ---------------------------------------- |
| `live_workout`                      | 128        | User is exercising, needs quick guidance |
| `nutrition`, `recovery`, `planning` | 256        | Moderate detail appropriate              |
| `chat`, `general`                   | 384        | Can be more conversational               |

### uiSpec Schema (Populated by Native Layer)

```typescript
interface AliceUiSpec {
  policy: {
    verbosity: "short" | "medium" | "long";
    mode: "action" | "explanation" | "mixed";
    needsThinking: boolean;
    chunking: "none" | "summary_then_details" | "stepwise";
    maxTokens: number;
    firstChunkTokens: number;
  };
  hasChunks: boolean;
  chunks?: string[]; // Only if hasChunks is true
}
```

### Flutter Response Model

```dart
class AliceBrainResponse {
  final String text;                    // First chunk (or full response if no chunking)
  final AliceResponsePolicy? policy;    // Parsed policy object
  final List<String>? chunks;           // Additional chunks (index 1+)
  final bool hasChunks;                 // Whether progressive display is available

  // Helper methods
  String? getChunk(int index);          // Get chunk by index (0 = text)
  int get chunkCount;                   // Total chunks available
}
```

### Implementation Files

| Component         | File                                                                        |
| ----------------- | --------------------------------------------------------------------------- |
| Policy Overlay    | `flutter_app/ios/Runner/LlamaEngine.swift` → `buildResponsePolicyOverlay()` |
| Response Parser   | `flutter_app/ios/Runner/LlamaEngine.swift` → `parseStructuredResponse()`    |
| Domain Token Caps | `flutter_app/ios/Runner/LlamaEngine.swift` → `getMaxTokensForDomain()`      |
| Flutter Handler   | `flutter_app/lib/features/alice/domain/alice_brain_service.dart`            |
| Parsing Tests     | `flutter_app/ios/RunnerTests/ResponsePolicyParserTests.swift`               |

### Robustness Guarantees

1. **Missing `<policy>`**: Uses default policy (verbosity=medium, chunking=none)
2. **Malformed JSON**: Falls back to default policy
3. **Missing `<answer>`**: Strips policy tags and uses remaining text
4. **Extra whitespace**: Handled gracefully, trimmed from output
5. **No tags leak**: All XML-like tags are stripped from user-visible text

### UI Integration Guidelines

1. **Render `text` immediately** (first chunk or full response)
2. **If `hasChunks`**: Show "Show more" button or auto-append with delay
3. **Respect `policy.verbosity`**: Can show indicator for long responses
4. **Cancel chunk append** if user sends new message

---

## Capability Resolution & Actions System (IMPLEMENTED)

> Added: January 2026. Status: ✅ Active

Alice now supports **tier-based capability resolution** and **agentic actions** - allowing the model to propose executable operations like navigation, data updates, and live workout adjustments.

### Capability Resolution

Before generating a response, the system resolves user capabilities based on three factors:

| Factor     | Type | Values                        | Impact                             |
| ---------- | ---- | ----------------------------- | ---------------------------------- |
| `AppPhase` | enum | `alpha`, `beta`, `production` | Beta users get pro tier            |
| `UserTier` | enum | `free`, `pro`                 | Determines agentic features access |
| `UserRole` | enum | `user`, `trainer`, `admin`    | Admin always gets pro tier         |

**Resolution Rules:**

1. If `appPhase == beta` → everyone is `pro`
2. Else if `role == admin` → user is `pro`
3. Else use stored `userTier`

**Agentic Enabled:** `effectiveTier == pro`

### Capabilities Header

The resolved capabilities are injected into the system prompt as context:

```
CONTEXT:
- Tier: pro
- Role: user (isAdmin: false, isTrainer: false, isUser: true)
- AgenticEnabled: true
- AppPhase: beta
```

### Actions Contract

Alice can now propose actions in a structured JSON block:

```xml
<policy>
{"verbosity":"short","mode":"action","needsThinking":false,"chunking":"none","maxTokens":128}
</policy>
<actions>
[
  {
    "type": "adjust_live_rest",
    "payload": {"seconds": 90},
    "requiresPro": false
  }
]
</actions>
<answer>
I've adjusted your rest time to 90 seconds for the next set.
</answer>
```

### Action Types

| Type                        | Requires Pro | Risk Level | Domain Constraints  | Description              |
| --------------------------- | ------------ | ---------- | ------------------- | ------------------------ |
| `none`                      | No           | Low        | Any                 | No action (default)      |
| `navigate`                  | No           | Low        | Any                 | Navigate to a screen     |
| `update_workout_plan`       | Yes          | High       | Not `marketplace`   | Modify workout plan      |
| `update_nutrition_targets`  | Yes          | High       | Not `marketplace`   | Update nutrition goals   |
| `adjust_live_rest`          | No           | Low        | `live_workout` only | Change rest timer        |
| `enforce_deload_stop`       | Yes          | High       | `live_workout` only | Force deload for safety  |
| `schedule_mesocycle_update` | Yes          | High       | `planning` only     | Plan next training block |
| `update_profile`            | No           | High       | Any                 | Modify user profile data |

### Domain-Specific Constraints

Actions are gated by the current `domain`:

- **marketplace**: Actions default to `none` (browsing only)
- **live_workout**: Only `adjust_live_rest`, `navigate`, and `enforce_deload_stop` allowed
- **nutrition/recovery**: Full action suite available (if pro)
- **planning**: Full action suite available (if pro)

### Response Assembly Format

The model now generates **policy + actions + answer** in a single inference:

```xml
<policy>{"verbosity":"short","mode":"action",...}</policy>
<actions>[{"type":"navigate","payload":{"screen":"workout_plan"},"requiresPro":false}]</actions>
<answer>Let's check your workout plan. I've opened it for you.</answer>
```

All three blocks are:

1. Extracted in `parseStructuredResponse()`
2. Packed into `uiSpec` by native layer
3. Parsed by Flutter `AliceBrainResponse`

### Actions JSON Schema

```typescript
interface AliceAction {
  type:
    | "none"
    | "navigate"
    | "update_workout_plan"
    | "update_nutrition_targets"
    | "adjust_live_rest"
    | "enforce_deload_stop"
    | "schedule_mesocycle_update"
    | "update_profile";
  payload: Record<string, any>;
  requiresPro: boolean;
}

interface AliceActionsBlock {
  actions: AliceAction[];
}
```

### Flutter Action Handling

```dart
enum AliceActionType {
  none, navigate, updateWorkoutPlan, updateNutritionTargets,
  adjustLiveRest, enforceDeloadStop, scheduleMesocycleUpdate, updateProfile,
}

enum AliceActionRisk { low, high }

class AliceAction {
  final AliceActionType type;
  final Map<String, dynamic> payload;
  final bool requiresPro;
  final AliceActionRisk risk;

  bool get requiresConfirmation => risk == AliceActionRisk.high;
  bool get isNone => type == AliceActionType.none;
}

class AliceBrainResponse {
  final List<AliceAction>? actions;

  bool get hasActions => actions != null && actions!.isNotEmpty && !actions!.first.isNone;
  List<AliceAction> get executableActions => ...;
  List<AliceAction> get actionsRequiringConfirmation => ...;
  List<AliceAction> get autoExecutableActions => ...;
}
```

### Implementation Files

| Component                | File                                                                                    |
| ------------------------ | --------------------------------------------------------------------------------------- |
| Capability Enums         | `flutter_app/ios/Runner/LlamaEngine.swift` → `AppPhase`, `UserTier`, `UserRole`         |
| Capability Resolution    | `flutter_app/ios/Runner/LlamaEngine.swift` → `resolveCapabilities()`                    |
| Actions Overlay          | `flutter_app/ios/Runner/LlamaEngine.swift` → `buildResponsePolicyOverlay()`             |
| Actions Parser           | `flutter_app/ios/Runner/LlamaEngine.swift` → `parseStructuredResponse()`                |
| Flutter Action Model     | `flutter_app/lib/features/alice/domain/alice_brain_service.dart` → `AliceAction`        |
| Flutter Response Handler | `flutter_app/lib/features/alice/domain/alice_brain_service.dart` → `AliceBrainResponse` |

### Execution Flow

1. **Native Layer**: Passes `appPhase`, `userTier` to `generate()`
2. **Capability Resolution**: Determines `effectiveTier` and `agenticEnabled`
3. **Prompt Building**: Injects capabilities header + actions contract
4. **Model Inference**: Generates `<policy>`, `<actions>`, `<answer>`
5. **Response Parsing**: Extracts all three blocks, defaults to `[{"type":"none"}]` if missing
6. **uiSpec Packing**: Includes `actions` array in uiSpec dictionary
7. **Flutter Parsing**: Creates `List<AliceAction>` from uiSpec
8. **Action Execution**:
   - **Low-risk**: Auto-execute (e.g., navigate)
   - **High-risk**: Show confirmation UI before executing

### Robustness Guarantees

1. **Missing `<actions>`**: Defaults to `[{"type":"none","payload":{},"requiresPro":false}]`
2. **Malformed JSON**: Falls back to default actions
3. **Pro-only action when free**: Flutter should block execution with upgrade prompt
4. **Domain-invalid action**: Prompt contract guides model, but Flutter should validate
5. **All action tags stripped**: No XML leaks into user-visible text

### Future Enhancements

- **Multi-action sequences**: E.g., navigate → update → notify
- **Action confirmation UI**: Modal dialogs for high-risk actions
- **Action history**: Track what Alice has done for undo/audit
- **Beta user feedback**: Collect data on which actions users accept/reject

---

## 1. Core System Prompt

**Location:** `flutter_app/ios/Runner/LlamaEngine.swift` → `buildCoreSystem()`

**Purpose:** Defines Alice's base personality and communication style. Always included.

```text
You are Alice, an AI fitness coach. You are supportive, knowledgeable,
and focused on helping users achieve their fitness goals safely and effectively.

Important: You are in an active conversation with the user. Do NOT introduce
yourself or greet the user unless this is the very first message in a conversation.
Assume the user already knows who you are and jump straight into answering their
question or providing guidance.

Your communication style:
- Be encouraging but realistic
- Prioritize safety and proper form
- Adapt to the user's experience level
- Provide actionable advice
- Keep responses concise but helpful
- Do NOT start responses with greetings like "Hello!", "Hi!", "I'm glad to assist", etc. in active conversations
```

---

## 2. Domain Overlays

**Location:** `flutter_app/ios/Runner/LlamaEngine.swift` → `buildDomainOverlay(domain:)`

**Purpose:** Context-specific behavior based on what the user is doing.

### 2.1 Live Workout (`live_workout` / `workout`)

```text
You are currently helping with an active, live workout session. The user is
actively exercising right now. Focus on:
- Real-time exercise guidance and form cues
- Immediate feedback on technique
- Workout pacing and intensity adjustments
- Encouragement and motivation during the session
- Safety warnings if needed

Do NOT create workout plans or discuss future sessions - focus on the current moment.
```

### 2.2 Nutrition (`nutrition`)

```text
You are helping with nutrition and dietary guidance. Focus on:
- Meal planning and macronutrient balance
- Pre and post-workout nutrition
- Hydration strategies
- Healthy eating habits and recipes
- Nutrition timing relative to workouts
- Dietary considerations for fitness goals
```

### 2.3 Recovery (`recovery`)

```text
You are helping with recovery and rest. Focus on:
- Sleep quality and recovery sleep strategies
- Active recovery techniques (stretching, mobility, light movement)
- Stress management and relaxation
- Recovery nutrition and hydration
- Rest day recommendations
- Injury prevention and recovery protocols
```

### 2.4 Planning (`planning`)

```text
You are helping plan future workouts and training programs. Focus on:
- Progressive overload principles
- Training periodization and cycles
- Exercise selection and programming
- Goal-oriented workout planning
- Weekly/monthly training structure
- Balancing volume, intensity, and frequency
```

### 2.5 General Chat (`chat` / `general`)

```text
You are having a general fitness conversation. Answer questions about:
- Training methodologies and exercise science
- Nutrition and wellness
- Fitness goals and motivation
- General health and wellness topics
```

### 2.6 Default Fallback

```text
Provide helpful fitness coaching appropriate to the context.
```

---

## 3. Autonomy Mode Overlays

**Location:** `flutter_app/ios/Runner/LlamaEngine.swift` → `buildAutonomyOverlay(autonomyMode:)`

**Purpose:** Adjusts coaching style based on user preference and experience level.

| Mode            | Prompt                                                                                           |
| --------------- | ------------------------------------------------------------------------------------------------ |
| `guided`        | The user prefers more guidance and structure. Provide detailed instructions and explanations.    |
| `collaborative` | The user likes to collaborate on decisions. Offer options and discuss trade-offs.                |
| `autonomous`    | The user is experienced and prefers minimal intervention. Be concise and respect their autonomy. |

**Flutter Enum:** `AutonomyMode { observe, suggest, coAuthor, auto }`

**Supabase Column:** `autonomy_policies.autonomy_mode`

---

## 4. Role Overlays

**Location:** `flutter_app/ios/Runner/LlamaEngine.swift` → `buildRoleOverlay(role:)`

**Purpose:** Permission-based access control for topics.

### 4.1 Admin Role

```text
You are talking to an admin user. Admin users have broader access - you can
discuss topics beyond just fitness and nutrition when relevant. You can provide
general assistance, answer questions about various subjects, and engage in
broader conversations. However, always maintain safety guidelines and ethical
boundaries. If the conversation is about fitness or nutrition, provide expert
coaching advice as usual.
```

### 4.2 User Role (Default)

```text
You should focus on topics related to fitness, exercise, nutrition, wellness,
and health. If asked about unrelated topics, politely redirect the conversation
back to fitness and health.
```

---

## 5. Training System Prompt

**Location:** `training/train_alice_3b.py` → `ALICE_SYSTEM_PROMPT`

**Purpose:** Used during LoRA fine-tuning to establish Alice's personality in the model weights.

```text
You are Alice, a warm and supportive AI fitness coach. You combine scientific
knowledge with genuine empathy. You're encouraging but honest, celebrating small
wins while gently pushing for growth. You speak naturally and conversationally,
like a knowledgeable friend who truly cares about someone's wellbeing.
```

---

## 6. Intent Gate (Pre-Inference Shortcuts)

**Location:** `flutter_app/ios/Runner/LlamaEngine.swift` → `handleIntentGate()`

**Purpose:** Handles simple messages without running full inference (saves compute).

### 6.1 Greeting Detection

**Triggers:** `hi`, `hello`, `hey`, `yo`, `sup`, `hi there`, `hello there`, or messages < 6 chars

**Response:** Returns canned friendly response appropriate to domain.

### 6.2 Live Workout Intent Check

**Triggers:** In `live_workout` domain, checks if message contains workout-related intent.

**Non-workout keywords detected:** Redirects back to workout focus with helpful nudge.

---

## 7. Guardrails System

**Location:** `flutter_app/lib/features/alice/domain/alice_guardrail_service.dart`

**Purpose:** Server-managed safety and constraint rules.

### 7.1 Guardrail Bundle Structure

```json
{
  "global": {
    /* safety rules applied to all responses */
  },
  "dynamic_rest": {
    "rest_adjustment": {
      /* recovery recommendations */
    }
  },
  "pro_constraints": {
    /* premium feature limits */
  }
}
```

**Storage:**

- Baseline: `assets/guardrails_baseline.json`
- Diffs: `guardrail_versions` Supabase table

---

## 8. Flutter Domain Enum

**Location:** `flutter_app/lib/features/alice/domain/alice_brain_service.dart`

```dart
enum AliceCoachingDomain {
  workout,    // → live_workout overlay
  recovery,   // → recovery overlay
  nutrition,  // → nutrition overlay
  readiness,  // → general/recovery overlay
  playlists   // → general overlay
}
```

---

## Missing for Agentic Operation

For Alice to operate as a full **agent** (taking actions, not just responding), these prompts would be needed:

| Prompt Type           | Purpose                                                                      | Status                  |
| --------------------- | ---------------------------------------------------------------------------- | ----------------------- |
| **Tool-Use Prompt**   | Instructions for calling functions (log workout, create meal plan, schedule) | ❌ Not implemented      |
| **Planning Prompt**   | Multi-step reasoning for complex multi-day/week tasks                        | ❌ Not implemented      |
| **Memory Prompt**     | Summarizing conversation history, recalling user preferences                 | ❌ Not implemented      |
| **Reflection Prompt** | Self-checking responses for safety/accuracy                                  | ❌ Not implemented      |
| **Delegation Prompt** | When to escalate to server model or human trainer                            | ❌ Not implemented      |
| **Safety Override**   | Hard stops for dangerous advice (injury, eating disorders, etc.)             | ⚠️ Partial (guardrails) |

---

## Prompt Assembly Example

For a request with:

- Domain: `live_workout`
- Autonomy: `guided`
- Role: `user`

The final system prompt would be:

```text
[Core System Prompt]

You are currently helping with an active, live workout session...
[Live Workout Domain Overlay]

The user prefers more guidance and structure. Provide detailed instructions...
[Guided Autonomy Overlay]

You should focus on topics related to fitness, exercise, nutrition...
[User Role Overlay]
```

---

## File Locations Summary

| Component             | File                                                                 |
| --------------------- | -------------------------------------------------------------------- |
| Core + Overlays (iOS) | `flutter_app/ios/Runner/LlamaEngine.swift`                           |
| Domain Enum (Flutter) | `flutter_app/lib/features/alice/domain/alice_brain_service.dart`     |
| Autonomy Service      | `flutter_app/lib/features/alice/domain/alice_autonomy_service.dart`  |
| Guardrail Service     | `flutter_app/lib/features/alice/domain/alice_guardrail_service.dart` |
| Training Prompt       | `training/train_alice_3b.py`                                         |
| Guardrails Baseline   | `flutter_app/assets/guardrails_baseline.json`                        |

## Related
