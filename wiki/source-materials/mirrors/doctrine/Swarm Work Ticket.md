# Swarm Work Ticket

## Concept
Each Swarm shard is executed under a Work Ticket to prevent scope creep.

## Rule / Mechanism
A Swarm Work Ticket must include:
- shard_id
- prompt_id / task_id reference
- shard objective
- allowed inputs (explicit)
- output schema
- max tokens / max time
- disallowed actions
- attribution metadata (device id, model id)

Nodes must refuse work outside the ticket.

## Why It Exists
Swarm parallelism increases surface area; tickets preserve safety and determinism.

## Implications
- Safe parallelism
- Clean debugging
- Predictable output contracts

## Links
- [[Swarm Task Sharding]]
- [[No Tool Access During Planning]]