---
title: Swarm Work Ticket
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Swarm Work Ticket.md"]
updated: 2026-07-24
---

# Swarm Work Ticket
[Swarm Work Ticket](https://www.notion.so/33ec72bad01381e29409d5ab37e5b618)
Concept
Each Swarm shard is executed under a Work Ticket to prevent scope creep.
Rule / Mechanism
A Swarm Work Ticket must include: - shard_id - prompt_id / task_id reference - shard objective - allowed inputs (explicit) - output schema - max tokens / max time - disallowed actions - attribution metadata (device id, model id)
Nodes must refuse work outside the ticket.
Why It Exists
Swarm parallelism increases surface area; tickets preserve safety and determinism.
Implications
Safe parallelism
Clean debugging
Predictable output contracts
Links
[Swarm Task Sharding](https://www.notion.so/33ec72bad013811e8b4ecb3b0ab21a89)
No Tool Access During Planning

## Related
