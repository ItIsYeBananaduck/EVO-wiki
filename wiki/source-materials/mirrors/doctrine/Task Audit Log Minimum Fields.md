# Task Audit Log Minimum Fields

## Concept
Task history must be reviewable with enough detail to understand what happened.

## Rule / Mechanism
For each executed task, the system must retain at minimum:
- Task ID, title, timestamps
- Authorization path (approved method vs Talent)
- Method or Talent reference
- Tool call log summary (what tools were used, not necessarily full payloads)
- Outcome (completed/failed/canceled)
- User-visible output artifacts

## Why It Exists
Without minimum fields, history becomes meaningless and trust erodes.

## Implications
- Consistent “what happened” UI
- Easier investigation of unexpected behavior
- Enables safe promotion decisions

## Links
- [[Task Transparency Retention]]
- [[Scoped Tool Grants]]
- [[Delegator Tool Hostage Rule]]