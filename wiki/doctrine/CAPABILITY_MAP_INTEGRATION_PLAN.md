---
title: CAPABILITY_MAP_INTEGRATION_PLAN
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/CAPABILITY_MAP_INTEGRATION_PLAN.md"]
updated: 2026-07-24
---

# Capability Map Integration Plan

## Overview

This plan outlines how to smartly utilize `alice_capability_map.json` for:

1. **Agentic Work**: Alice referencing capabilities when making decisions and taking actions
2. **Admin Queries**: Alice answering questions about her capabilities, access rules, and functionality

## Current State

- ✅ **Capability Map Exists**: Comprehensive JSON with 37 capabilities (17 agentic actions, 9 automatic actions, 5 vision capabilities, 6 trainer capabilities)
- ✅ **Well-Structured**: Each capability has `whatItDoes`, `whenToUse`, `howToUse`, `access`, `payload`, `examples`
- ❌ **Not Integrated**: Map is not currently loaded or referenced in Alice's inference pipeline
- ❌ **No Admin Interface**: No way for admins to query Alice about capabilities

## Important: Two Different Capability Systems

### 1. CAPABILITIES_JSON (Runtime Config)

**What it is**: Compact JSON generated at runtime in `LlamaEngine.swift`
**Format**: `{"tier":"free","role":"user","agenticEnabled":false,"canUseAiPlanOps":false,...}`
**Purpose**: Runtime permissions/config - tells Alice what the user CAN do
**Where**: Injected as `INTERNAL_CONFIG:` in system prompt
**ENF Training**: ENF model is trained to follow this format

### 2. alice_capability_map.json (Reference Guide)

**What it is**: Comprehensive capability reference with actions, when to use, how to use
**Format**: Detailed JSON with `whatItDoes`, `whenToUse`, `howToUse`, `payload`, `examples`
**Purpose**: Reference guide - tells Alice what actions exist and how to use them
**Where**: Should be loaded and filtered, then injected into system prompt
**ENF Training**: ENF should be trained to reference this for action selection

### How They Work Together

```
CAPABILITIES_JSON (Runtime Config)
  ↓
  Tells Alice: "User is free tier, agentic disabled, can't use AI plan ops"
  ↓
alice_capability_map.json (Reference Guide)
  ↓
  Tells Alice: "update_workout_plan exists, requires Pro, here's how to use it"
  ↓
Alice Decision:
  - Sees update_workout_plan in capability map
  - Checks CAPABILITIES_JSON: agenticEnabled=false, tier=free
  - Decides: Can't use this action (blocked by runtime config)
```

**Alice does NOT access the file directly**. Instead:

1. **System loads the file** - `LlamaEngine` loads `alice_capability_map.json` at runtime
2. **System filters by context** - Only relevant capabilities are included (domain, tier, role)
3. **System injects into prompt** - Filtered capabilities are added alongside `CAPABILITIES_JSON`
4. **Alice uses both** - Alice uses `CAPABILITIES_JSON` for permissions, capability map for action details

---

## Phase 1: Agentic Work Integration

### Goal

Enable Alice to reference the capability map when deciding which actions to take, ensuring she:

- Knows what actions are available
- Understands when to use each action
- Follows correct payload structures
- Respects access rules

### Implementation Strategy

#### 1.1 Dynamic Capability Loading

**Location**: `flutter_app/ios/Runner/LlamaEngine.swift` (or equivalent Android)

**Approach**: Load and filter capabilities based on context

```swift
// Pseudo-code structure
func loadRelevantCapabilities(
    domain: String,
    userTier: String,
    isAdmin: Bool,
    isTrainer: Bool,
    agenticEnabled: Bool
) -> [String: Any] {
    // Load capability map JSON
    let fullMap = loadCapabilityMap()

    // Filter based on context
    var relevant: [String: Any] = [:]

    // Always include agentic actions
    relevant["agenticActions"] = filterByAccess(
        fullMap["agenticActions"],
        requiresPro: userTier == "pro",
        agenticEnabled: agenticEnabled
    )

    // Include automatic actions if in workout context
    if domain == "live_workout" {
        relevant["automaticActions"] = fullMap["automaticActions"]
    }

    // Include trainer capabilities if user is trainer
    if isTrainer {
        relevant["trainerCapabilities"] = fullMap["trainerCapabilities"]
    }

    // Include admin capabilities if admin
    if isAdmin {
        relevant["adminCapabilities"] = fullMap["adminCapabilities"]
    }

    return relevant
}
```

#### 1.2 System Prompt Integration

**Location**: `buildSystemPrompt()` in `LlamaEngine.swift`

**Approach**: Inject relevant capability sections into system prompt

```swift
func buildSystemPrompt(...) -> String {
    // ... existing prompt layers ...

    // NEW: Capability Map Layer
    let relevantCapabilities = loadRelevantCapabilities(...)
    let capabilityContext = formatCapabilityContext(relevantCapabilities)

    return """
    \(coreSystem)
    \(domainOverlay)
    \(autonomyOverlay)
    \(roleOverlay)

    ## CAPABILITY MAP

    You have access to the following actions. Consult this map when deciding what to do:

    \(capabilityContext)

    When performing actions:
    1. Check the 'whenToUse' array to confirm this is the right action
    2. Verify 'access' rules (requiresPro, agenticEnabled)
    3. Follow 'howToUse' step-by-step instructions
    4. Construct payload according to 'payload' structure
    5. Format action in <actions> tag with proper JSON
    """
}
```

#### 1.3 Context-Aware Filtering

**Strategy**: Only include capabilities relevant to current context

**Filtering Rules**:

- **Domain-based**: Only include actions relevant to current domain (workout/nutrition/recovery)
- **Tier-based**: Filter out Pro-only actions for free users
- **Autonomy-based**: Include automatic actions only when in 'auto' mode
- **Role-based**: Include admin/trainer capabilities only for those roles

**Example**:

```swift
// In workout domain, only include:
- update_workout_plan
- adjust_live_rest
- enforce_deload_stop
- schedule_mesocycle_update
- Automatic actions (strain-based rest, progressions, etc.)

// Exclude:
- update_nutrition_targets (nutrition domain)
- Vision capabilities (unless workout uses pose estimation)
```

#### 1.4 Token Optimization

**Challenge**: Full capability map is ~1782 lines - too large for every prompt

**Solutions**:

**Option A: Selective Inclusion** (Recommended)

- Include only relevant capabilities based on context
- Use compact format (remove examples, keep structure)
- Estimated: 200-500 tokens per context

**Option B: Summarized Format**

- Create a condensed version with just:
  - Action name
  - When to use (one line)
  - Access rules
  - Payload structure (minimal)
- Estimated: 100-300 tokens per context

**Option C: On-Demand Reference**

- Include capability map in training data so Alice learns it
- Only include trigger: "Consult your capability map for [action type]"
- Estimated: 20-50 tokens per context

**Recommendation**: **Option A** (Selective Inclusion) - best balance of completeness and efficiency

---

## Phase 2: Admin Query Interface

### Goal

Enable admins to ask Alice questions about her capabilities and get accurate answers

### Implementation Strategy

#### 2.1 Admin Query Detection

**Location**: `buildSystemPrompt()` - Role Overlay

**Approach**: Detect admin queries and enable capability Q&A mode

```swift
func buildSystemPrompt(role: String, userMessage: String, ...) -> String {
    let isAdmin = role == "admin"

    // Detect admin capability queries
    let isCapabilityQuery = isAdmin && detectCapabilityQuery(userMessage)

    if isCapabilityQuery {
        return buildAdminCapabilityPrompt(userMessage)
    }

    // ... normal prompt ...
}
```

**Query Detection Patterns**:

- "What can you do?"
- "What actions are available?"
- "How do I use [action]?"
- "What are the access rules for [action]?"
- "List all capabilities"
- "Show me the capability map"

#### 2.2 Admin Capability Prompt

**Structure**: Full capability map + Q&A instructions

```swift
func buildAdminCapabilityPrompt(query: String) -> String {
    let fullMap = loadFullCapabilityMap() // No filtering for admins

    return """
    You are Alice, an AI fitness coach. An admin is asking about your capabilities.

    ## FULL CAPABILITY MAP

    \(formatFullCapabilityMap(fullMap))

    ## ADMIN INSTRUCTIONS

    Answer the admin's question about your capabilities. You can:
    - List all available actions
    - Explain what a specific action does
    - Describe when to use an action
    - Show access rules and requirements
    - Provide payload examples
    - Explain automatic action triggers

    Be thorough and accurate. Reference the capability map directly.

    Admin Query: \(query)
    """
}
```

#### 2.3 Capability Search Function

**Enhancement**: Allow Alice to search/filter capabilities in response

**Query Types**:

1. **List All**: "What actions can you take?"
2. **By Domain**: "What can you do in workouts?"
3. **By Access**: "What actions require Pro?"
4. **Specific Action**: "How does update_workout_plan work?"
5. **Trigger Questions**: "When does strain-based rest adjustment trigger?"

**Implementation**:

```swift
func searchCapabilities(query: String, map: [String: Any]) -> [String: Any] {
    // Parse query intent
    if query.contains("all") || query.contains("list") {
        return map // Return everything
    }

    if query.contains("workout") {
        return filterByDomain(map, domain: "workout")
    }

    if query.contains("require") && query.contains("pro") {
        return filterByAccess(map, requiresPro: true)
    }

    // Extract action name from query
    if let actionName = extractActionName(query) {
        return getCapability(map, action: actionName)
    }

    return map
}
```

#### 2.4 Admin UI Integration

**Location**: Admin dashboard or settings screen

**Features**:

1. **Capability Browser**: View all capabilities in organized UI
2. **Query Interface**: Chat with Alice about capabilities
3. **Search**: Search capabilities by name, domain, access level
4. **Export**: Export capability map as JSON/Markdown

**Example UI Flow**:

```
Admin Dashboard → "Alice Capabilities" → Query Interface
  ↓
Admin: "What actions require Pro tier?"
  ↓
Alice: Lists all Pro-required actions with details
  ↓
Admin: "How does update_workout_plan work?"
  ↓
Alice: Explains action, when to use, payload structure, examples
```

---

## Phase 3: Training Data Integration

### Goal

Ensure Alice learns to reference and use the capability map through training

### Implementation Strategy

#### 3.1 Add Capability Map References to Training Data

**Location**: `training/enf_lora/data/enf_train.jsonl`

**Approach**: Add examples where Alice references the capability map

**Example Training Entry**:

```json
{
  "instruction": "User: Create a push/pull/legs workout plan",
  "input": "",
  "output": "<thinking>User wants a new workout plan. Consulting capability map for update_workout_plan action:\n- Action: update_workout_plan\n- Access: requiresPro=true, agenticEnabled=true (user has Pro)\n- When to use: User requests new workout plan\n- Payload: omit planId for new plan, include schedule type and daysPerWeek\n- Confirmation: requiresConfirmation=true, so I'll inform user</thinking>\n\n<actions>{\"actions\": [{\"type\": \"update_workout_plan\", \"payload\": {\"changes\": {\"schedule\": {\"type\": \"push_pull_legs\", \"daysPerWeek\": 6}, \"notes\": \"User requested new push/pull/legs plan\"}}, \"requiresPro\": true, \"requiresConfirmation\": true}]}</actions>\n\n<answer>I'll create a push/pull/legs plan for you! This will be a 6-day split. Let me set that up - I'll need your confirmation before applying it.</answer>"
}
```

#### 3.2 Add Admin Query Examples

**Training Entries for Admin Queries**:

```json
{
  "instruction": "Admin: What actions can you take?",
  "input": "",
  "output": "<answer>I have access to 17 agentic actions:\n\n1. **navigate** - Screen navigation (no Pro required)\n2. **update_workout_plan** - Create/update workout plans (requires Pro)\n3. **update_nutrition_targets** - Set nutrition goals (requires Pro)\n4. **adjust_live_rest** - Rest time adjustments (no Pro required)\n5. **enforce_deload_stop** - Safety deloads (requires Pro)\n6. **schedule_mesocycle_update** - Schedule updates (requires Pro)\n7. **update_profile** - Profile changes (no Pro required)\n8. **none** - No action needed\n\nPlus 9 automatic actions that trigger based on system conditions, 5 vision capabilities, and 6 trainer capabilities. Would you like details on any specific action?</answer>"
}
```

#### 3.3 Capability Map Lookup Examples

**Training Entries Showing Map Consultation**:

```json
{
  "instruction": "System: Strain score 97% detected during rest period. System requesting automatic rest extension.",
  "input": "",
  "output": "<thinking>System triggered automatic action. Consulting capability map for adjust_live_rest:\n- Action: adjust_live_rest\n- Automatic trigger: strainBasedRestAdjustment\n- Trigger condition: Strain >95% (red zone)\n- Access: requiresPro=false, autoExecute=true\n- Payload: restSeconds (extend significantly), reason, strainScore, strainStatus\n- No confirmation needed (autoExecute)</thinking>\n\n<actions>{\"actions\": [{\"type\": \"adjust_live_rest\", \"payload\": {\"restSeconds\": 120, \"reason\": \"Strain score 97% (red zone) - extending rest significantly\", \"strainScore\": 97, \"strainStatus\": \"red\"}, \"requiresPro\": false, \"autoExecute\": true}]}</actions>\n\n<answer>Extending your rest period - your strain is in the red zone at 97%. Take an extra minute to recover.</answer>"
}
```

---

## Phase 4: Optimization & Performance

### 4.1 Capability Map Caching

**Strategy**: Cache parsed capability map in memory

```swift
class CapabilityMapCache {
    static var shared = CapabilityMapCache()
    private var cachedMap: [String: Any]?
    private var lastLoadTime: Date?

    func getMap() -> [String: Any]? {
        // Reload if older than 1 hour or first load
        if cachedMap == nil ||
           lastLoadTime == nil ||
           Date().timeIntervalSince(lastLoadTime!) > 3600 {
            loadMap()
        }
        return cachedMap
    }

    private func loadMap() {
        // Load from JSON file
        // Cache in memory
    }
}
```

### 4.2 Lazy Loading

**Strategy**: Only load capabilities when needed

- **Agentic Work**: Load on first action request
- **Admin Query**: Load on admin query detection
- **Training**: Pre-load for training examples

### 4.3 Format Optimization

**Strategy**: Use compact format for system prompts

**Compact Format**:

```json
{
  "update_workout_plan": {
    "when": ["user requests plan", "week progression", "mesocycle transition"],
    "access": {"pro": true, "confirm": true},
    "payload": {"planId?": "string", "changes": {...}}
  }
}
```

**Full Format** (for admin queries):

- Keep full format with examples, detailed instructions

---

## Implementation Checklist

### Phase 1: Agentic Work Integration

- [ ] Create `CapabilityMapLoader` class in native code
- [ ] Implement context-aware filtering
- [ ] Integrate into `buildSystemPrompt()`
- [ ] Add capability context to system prompt
- [ ] Test with sample agentic actions
- [ ] Verify token usage stays reasonable (<500 tokens)

### Phase 2: Admin Query Interface

- [ ] Implement admin query detection
- [ ] Create `buildAdminCapabilityPrompt()` function
- [ ] Add capability search/filter functions
- [ ] Test with various admin queries
- [ ] Create admin UI (optional, can be chat-based)

### Phase 3: Training Data Integration

- [ ] Generate training examples with capability map references
- [ ] Add admin query examples
- [ ] Add automatic action trigger examples
- [ ] Validate examples follow capability map structure
- [ ] Add to `enf_train.jsonl`

### Phase 4: Optimization

- [ ] Implement capability map caching
- [ ] Create compact format for system prompts
- [ ] Measure token usage impact
- [ ] Optimize filtering logic
- [ ] Add telemetry for capability map usage

---

## Testing Strategy

### Unit Tests

1. **Capability Loading**: Test loading and parsing JSON
2. **Filtering**: Test context-based filtering
3. **Query Detection**: Test admin query pattern matching
4. **Search**: Test capability search functions

### Integration Tests

1. **Agentic Actions**: Test Alice using capability map for actions
2. **Admin Queries**: Test Alice answering capability questions
3. **Token Usage**: Verify prompts stay within limits
4. **Performance**: Measure load time and memory usage

### User Testing

1. **Agentic Work**: Have users request actions, verify Alice uses correct capabilities
2. **Admin Queries**: Have admins ask various capability questions
3. **Edge Cases**: Test with missing capabilities, invalid queries

---

## Success Metrics

### Agentic Work

- ✅ Alice correctly identifies appropriate actions from capability map
- ✅ Alice follows correct payload structures
- ✅ Alice respects access rules (Pro, agenticEnabled)
- ✅ Token usage stays under 500 tokens per prompt

### Admin Queries

- ✅ Alice accurately answers capability questions
- ✅ Alice references capability map in responses
- ✅ Admin can discover all available actions
- ✅ Response time < 2 seconds for admin queries

---

## Future Enhancements

1. **Capability Versioning**: Track capability map versions, notify when updated
2. **Usage Analytics**: Track which capabilities are used most
3. **Auto-Documentation**: Generate docs from capability map
4. **Capability Testing**: Automated tests for each capability
5. **Dynamic Updates**: Update capability map without app update (remote config)

---

## Files to Modify

### Native Code

- `flutter_app/ios/Runner/LlamaEngine.swift` - Add capability loading and prompt integration
- `flutter_app/android/.../LlamaEngine.kt` - Android equivalent

### Training Data

- `training/enf_lora/data/enf_train.jsonl` - Add capability map examples

### Documentation

- `training/enf_lora/CAPABILITY_MAP_USAGE.md` - Update with integration details
- `docs/ALICE_SYSTEM_PROMPTS.md` - Document capability map layer

### Assets

- `training/enf_lora/alice_capability_map.json` - Keep updated with new capabilities

---

## Questions to Resolve

1. **Token Budget**: What's the maximum token budget for capability map in system prompt?
2. **Update Frequency**: How often will capability map change? (affects caching strategy)
3. **Admin UI**: Should we build a dedicated admin UI or use chat interface?
4. **Training Priority**: Should we prioritize training data or system prompt integration?
5. **Fallback**: What happens if capability map fails to load?

---

## Next Steps

1. **Review & Approve Plan**: Get stakeholder approval
2. **Start Phase 1**: Implement agentic work integration (highest priority)
3. **Test Phase 1**: Validate with real agentic actions
4. **Start Phase 2**: Implement admin query interface
5. **Generate Training Data**: Create examples for Phase 3
6. **Optimize**: Implement Phase 4 optimizations based on real usage

---

**Created**: 2025-01-15
**Status**: Draft - Ready for Review
**Owner**: AI Development Team

## Related

^[{src_rel}]
