---
title: Offline Lease Holder Rule
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Offline Lease Holder Rule.md
updated: 2026-07-24
---

# Offline Lease Holder Rule
[Offline Lease Holder Rule](https://www.notion.so/33ec72bad01381a6a2d8dfc3ff49d7c8)
Concept
The lease holder device may operate offline.
Rule / Mechanism
If the lease holder is offline: - it may still run local inference - it may still enforce Delegator rules - it may still accept prompts and create tasks
However: - Hive/Swarm delegation is unavailable until connectivity returns - state sync must reconcile when reconnected
Why It Exists
Users should be able to use Alice without requiring connectivity on their active device.
Implications
Local-first behavior remains functional
Deferred sync is required
Links
[Execution Lease Rule](https://www.notion.so/33ec72bad01381d7846bf9f2cabb72fd)
[Hive Shared State Backbone](https://www.notion.so/33ec72bad01381a2845acb86cd36ef48)

## Related

^[source-materials/mirrors/doctrine/Offline Lease Holder Rule.md]
