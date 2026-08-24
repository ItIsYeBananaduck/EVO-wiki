---
title: ACTION_CARD_QUERY
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/ACTION_CARD_QUERY.md"]
updated: 2026-07-24
---

# Action Card Query System

**Problem**: Loading entire capability map into context uses 2000+ tokens. For 100 actions, this doesn't scale.

**Solution**: Alice queries specific action cards on-demand instead of loading the full map.

## Architecture

```
Traditional:  User → System loads full map (2000 tokens) → Alice finds action → Formats
Action Cards: User → Alice detects action need → Queries action card (100 tokens) → Formats
```

## Flow

1. **Alice detects action need**: "User wants navigation"
2. **Alice queries action card**: `getActionCard("navigate")`
3. **System returns just that card**: Navigate definition (~100 tokens)
4. **Alice formats action**: Uses card's payload structure

## Benefits

| Metric             | Full Map                    | Action Cards           |
| ------------------ | --------------------------- | ---------------------- |
| Tokens per request | 2000+                       | ~100                   |
| Scalability        | Linear (N actions = N×2000) | Constant (always ~100) |
| Speed              | Parse full JSON             | Direct lookup          |
| Context window     | Large                       | Minimal                |

## Implementation

### 1. Action Card Query Function

```swift
/// Get a specific action card from capability map (on-demand query)
/// Returns just that action's definition - much smaller than full map
@objc func getActionCard(_ actionName: String) -> [String: Any]? {
    // Load capability map (cached)
    guard let fullMap = loadRelevantCapabilities(...) else { return nil }

    // Extract just this action
    if let agenticActions = fullMap["agenticActions"] as? [String: Any],
       let actionCard = agenticActions[actionName] as? [String: Any] {
        return actionCard
    }

    return nil
}
```

### 2. Update Meta-LoRA Training

Teach Alice to:

- Detect when she needs an action
- Call `getActionCard()` to query the card
- Use the card to format the action

### 3. System Prompt Update

Instead of:

```
CAPABILITY_MAP: {entire map with 17 actions}
```

Use:

```
ACTION_CARD_QUERY available: Call getActionCard("action_name") to get action definition.
When user requests an action:
1. Detect which action is needed
2. Query that action's card
3. Use card's payload structure to format action
```

## Token Savings

| Scenario         | Full Map    | Action Cards | Savings |
| ---------------- | ----------- | ------------ | ------- |
| 1 action needed  | 2000 tokens | 100 tokens   | **95%** |
| 5 actions needed | 2000 tokens | 500 tokens   | **75%** |
| 17 actions       | 2000 tokens | 1700 tokens  | **15%** |
| 100 actions      | 2000 tokens | 100 tokens   | **95%** |

**Key insight**: Action cards scale linearly with usage, not with total actions.

## When to Use Full Map vs Cards

**Full Map**:

- Admin queries about all capabilities
- System debugging
- Initial setup/training

**Action Cards**:

- User requests specific action (99% of cases)
- Lower context window priority
- Better scalability

## Example

**User**: "Take me to my workouts"

**Alice's thinking**:

```
1. User wants navigation → need "navigate" action
2. Query action card: getActionCard("navigate")
3. Card returned: {"payload": {"route": "string"}, "howToUse": {...}}
4. Format action: {"type": "navigate", "payload": {"route": "/workouts"}}
```

**Tokens used**: ~100 (just navigate card) vs 2000 (full map)

## Implementation Priority

1. ✅ Add `getActionCard()` function to LlamaEngine
2. ✅ Update meta-LoRA training data to teach querying
3. ✅ Update system prompt to mention ACTION_CARD_QUERY
4. ⏳ Test: Does Alice learn to query cards automatically?

## Related
