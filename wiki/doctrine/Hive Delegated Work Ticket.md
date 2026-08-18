---
title: Hive Delegated Work Ticket
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Hive Delegated Work Ticket.md"]
updated: 2026-07-24
---

# Hive Delegated Work Ticket
[Hive Delegated Work Ticket](https://www.notion.so/33ec72bad01381f3b089ec4f59e001ab)
Concept
Delegated work must be bound to an explicit ticket to prevent scope creep.
Rule / Mechanism
A Work Ticket includes: - task_id / prompt_id - allowed work type (e.g., summarize, draft method, heavy inference) - max tokens / max time - allowed inputs (chat excerpt range, task metadata) - disallowed actions (no tools, no secrets, no execution) - expected output schema (artifact type)
The delegated device must refuse work outside the ticket.
Why It Exists
Delegation is a security boundary: tickets prevent silent expansion of work or authority.
Implications
Deterministic delegation
Easy auditing
Prevents “helpful” overreach from models
Links
Swarm Agents Are Non-Executable
No Tool Access During Planning

## Related

^[{src_rel}]
