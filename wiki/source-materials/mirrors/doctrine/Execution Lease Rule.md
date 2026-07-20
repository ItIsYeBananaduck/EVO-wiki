# Execution Lease Rule

## Concept
Only one Hive member may execute tasks at any given time.

## Rule / Mechanism
The execution lease:
- is held by the device the user is actively using
- grants the ability to execute authorized tasks
- must be released before another device can execute

Devices without the lease may not execute tools.

## Why It Exists
Prevents race conditions, duplicate actions, and conflicting side effects.

## Implications
- Safe device switching
- Deterministic execution
- Clear responsibility

## Links
- [[Delegator Tool Hostage Rule]]
- [[Single Executor Guarantee]]