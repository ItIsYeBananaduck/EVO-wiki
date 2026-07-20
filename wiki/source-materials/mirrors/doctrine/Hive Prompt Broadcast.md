# Hive Prompt Broadcast

## Concept
Prompts created on any device (in any EVO app) are visible to the entire Hive.

## Rule / Mechanism
When a user submits a prompt:
- it is appended to the shared Hive chat stream
- all Hive members receive the prompt event
- only the lease holder may initiate execution

## Why It Exists
Enables seamless multi-device continuity and supports compute delegation.

## Implications
- Same chat everywhere
- Lease holder remains the single executor gate
- Non-lease devices can still contribute via planning/bidding

## Links
- [[Hive Definition]]
- [[Execution Lease Rule]]
- [[Hive Read-Only Members]]