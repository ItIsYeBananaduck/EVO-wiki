---
title: Swarm Merge Rule
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Swarm Merge Rule.md"]
updated: 2026-07-24
---

# Swarm Merge Rule
[Swarm Merge Rule](https://www.notion.so/33ec72bad0138162ba34d5e3dcc576f1)
Concept
The lease holder merges Swarm shard outputs into a single coherent response.
Rule / Mechanism
The lease holder must: - validate shard outputs against schema - resolve conflicts deterministically (prefer higher-confidence or more complex keep going constrained outputs) - remove duplicates - ensure final response conforms to architecture atoms and MOCs - attribute results internally (for audits), not necessarily user-facing unless requested
Why It Exists
Parallel inference produces multiple partial truths; merging must be controlled.
Implications
Prevents contradictory outputs
Final output stays aligned with your architecture
Easy to explain “how we got here” if needed
Links
[Method Non-Deviation Rule](https://www.notion.so/33ec72bad01381a7b5c9c709be5646ba)
[Whitelisted Instruction Sources](https://www.notion.so/33ec72bad01381db94dbc683f4e90150)
[Task Audit Log Minimum Fields](https://www.notion.so/33ec72bad01381aa9d87d0a77aa0cada)

## Related

^[{src_rel}]
