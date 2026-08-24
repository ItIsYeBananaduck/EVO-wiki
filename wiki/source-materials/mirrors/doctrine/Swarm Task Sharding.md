---
title: Swarm Task Sharding
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Swarm Task Sharding.md"]
updated: 2026-07-24
---

# Swarm Task Sharding
[Swarm Task Sharding](https://www.notion.so/33ec72bad013811e8b4ecb3b0ab21a89)
Concept
Heavy inference is decomposed into bounded inference sub-tasks.
Rule / Mechanism
The lease holder must shard work into tasks with: - clear objective - explicit input slice (chat range, relevant artifacts) - output schema (structured result) - token/time budget - disallowed actions (no tools, no secrets unless explicitly permitted)
Examples of shard types: - extract requirements - generate method draft - risk analysis - summarization - alternative solution generation - contradiction detection - UI copy drafting - acceptance criteria drafting
Why It Exists
Parallel work must be bounded and composable.
Implications
Shards are auditable and reproducible
Results can be merged deterministically
Links
[Swarm Parallel Inference](https://www.notion.so/33ec72bad013810eaa8ac7df235f10d0)
[Swarm Merge Rule](https://www.notion.so/33ec72bad0138162ba34d5e3dcc576f1)

## Related
