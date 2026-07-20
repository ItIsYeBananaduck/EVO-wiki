# Swarm Shard Retry Policy

## Concept
Swarm shards may fail due to transient device constraints and must be handled safely.

## Rule / Mechanism
Shard failure handling:
- If a shard fails due to device unavailability, retry on a different eligible device
- If a shard fails due to timeout, retry once with reduced scope or budget
- If a shard fails repeatedly, mark as failed and continue merge with partial results

Retry limits:
- Maximum retries per shard: configurable (default: 1)
- Infinite retries are forbidden

Shard failure must never:
- block the entire Swarm indefinitely
- trigger execution
- escalate privileges

## Why It Exists
Distributed inference requires resilience without runaway behavior.

## Implications
- Graceful degradation
- No deadlocks
- Partial intelligence is preferred over none

## Links
- [[Swarm Work Ticket]]
- [[Swarm Merge Rule]]
- [[Single Executor Guarantee]]