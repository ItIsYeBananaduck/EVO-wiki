# Offline Lease Holder Rule

## Concept
The lease holder device may operate offline.

## Rule / Mechanism
If the lease holder is offline:
- it may still run local inference
- it may still enforce Delegator rules
- it may still accept prompts and create tasks

However:
- Hive/Swarm delegation is unavailable until connectivity returns
- state sync must reconcile when reconnected

## Why It Exists
Users should be able to use Alice without requiring connectivity on their active device.

## Implications
- Local-first behavior remains functional
- Deferred sync is required

## Links
- [[Execution Lease Rule]]
- [[Hive Shared State Backbone]]