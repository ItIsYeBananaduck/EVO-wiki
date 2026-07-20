# Shard-to-Device Assignment Heuristics

## Concept
Swarm shards are assigned to Hive devices based on capability, safety, and efficiency.

## Rule / Mechanism
The lease holder assigns shards using deterministic heuristics derived from each device’s capability profile.

Primary assignment criteria:
1) Ability to complete the shard within constraints (hard gate)
2) Thermal headroom
3) Power state (prefer charging devices)
4) Model readiness (warm > cold)
5) Expected latency vs shard urgency
6) Device class suitability (desktop > tablet > phone for heavy shards)

Shard-type preferences:
- Heavy reasoning shards → highest compute, best thermal margin
- Risk / consistency shards → reliable, stable devices
- Summarization shards → any capable device
- UI copy shards → lower priority, lighter devices

## Why It Exists
Prevents inefficient or unsafe shard placement and avoids user-visible slowdowns.

## Implications
- Predictable Swarm behavior
- Better battery and thermal outcomes
- Easier debugging and tuning

## Links
- [[Hive Capability Advertisement]]
- [[Swarm Task Sharding]]
- [[Swarm Parallel Inference]]