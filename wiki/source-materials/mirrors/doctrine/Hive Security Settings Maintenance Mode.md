# Hive Security Settings Maintenance Mode

## Concept
Trust and device-management actions require a safe configuration mode.

## Rule / Mechanism
When the user enters Hive Security Settings:
- local inference pauses
- all tool grants are revoked
- swarm and delegation are disabled
- only configuration actions are allowed

When the user exits:
- normal operation resumes

## Why It Exists
Prevents side effects during trust changes and keeps state transitions atomic.

## Implications
- No mid-execution confusion
- Lower risk of inconsistent authority
- Clear user intent

## Links
- [[Delegator Tool Hostage Rule]]
- [[Execution Lease Rule]]