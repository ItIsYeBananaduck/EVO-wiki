---
title: Shard-to-Device Assignment Heuristics
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Shard-to-Device Assignment Heuristics.md"]
updated: 2026-07-24
---

# Shard-to-Device Assignment Heuristics
[Shard-to-Device Assignment Heuristics](https://www.notion.so/33ec72bad01381ffa3ddef1fdccbecaa)
Concept
Swarm shards are assigned to Hive devices based on capability, safety, and efficiency.
Rule / Mechanism
The lease holder assigns shards using deterministic heuristics derived from each device’s capability profile.
Primary assignment criteria: 1) Ability to complete the shard within constraints (hard gate) 2) Thermal headroom 3) Power state (prefer charging devices) 4) Model readiness (warm > cold) 5) Expected latency vs shard urgency 6) Device class suitability (desktop > tablet > phone for heavy shards)
Shard-type preferences: - Heavy reasoning shards → highest compute, best thermal margin - Risk / consistency shards → reliable, stable devices - Summarization shards → any capable device - UI copy shards → lower priority, lighter devices
Why It Exists
Prevents inefficient or unsafe shard placement and avoids user-visible slowdowns.
Implications
Predictable Swarm behavior
Better battery and thermal outcomes
Easier debugging and tuning
Links
[Hive Capability Advertisement](https://www.notion.so/33ec72bad0138187a984facd654b00d9)
[Swarm Task Sharding](https://www.notion.so/33ec72bad013811e8b4ecb3b0ab21a89)
[Swarm Parallel Inference](https://www.notion.so/33ec72bad013810eaa8ac7df235f10d0)

## Related
