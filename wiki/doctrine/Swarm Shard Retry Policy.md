---
title: Swarm Shard Retry Policy
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Swarm Shard Retry Policy.md"]
updated: 2026-07-24
---

# Swarm Shard Retry Policy
[Swarm Shard Retry Policy](https://www.notion.so/33ec72bad01381ccbcf9e8684192bd67)
Concept
Swarm shards may fail due to transient device constraints and must be handled safely.
Rule / Mechanism
Shard failure handling: - If a shard fails due to device unavailability, retry on a different eligible device - If a shard fails due to timeout, retry once with reduced scope or budget - If a shard fails repeatedly, mark as failed and continue merge with partial results
Retry limits: - Maximum retries per shard: configurable (default: 1) - Infinite retries are forbidden
Shard failure must never: - block the entire Swarm indefinitely - trigger execution - escalate privileges
Why It Exists
Distributed inference requires resilience without runaway behavior.
Implications
Graceful degradation
No deadlocks
Partial intelligence is preferred over none
Links
[Swarm Work Ticket](https://www.notion.so/33ec72bad01381e29409d5ab37e5b618)
[Swarm Merge Rule](https://www.notion.so/33ec72bad0138162ba34d5e3dcc576f1)
[Single Alice Illusion](https://www.notion.so/33ec72bad01381ea8395c1c9c35f9066)

## Related

^[{src_rel}]
