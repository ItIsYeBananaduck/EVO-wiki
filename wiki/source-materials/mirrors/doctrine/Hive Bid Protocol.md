---
title: Hive Bid Protocol
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Hive Bid Protocol.md"]
updated: 2026-07-24
---

# Hive Bid Protocol
[Hive Bid Protocol](https://www.notion.so/33ec72bad01381248178cdbbb777e767)
Concept
If the lease holder cannot (or should not) run an inference/task, it may offer the work to the rest of the Hive to bid.
Rule / Mechanism
Lease holder emits a Bid Request containing:
task_id / prompt_id
required work type (plan / summarize / heavy inference / tool-mapping)
estimated token budget
urgency (interactive vs background-acceptable)
required constraints (no tools, tools, secrets, etc.)
Hive members respond with bids describing:
expected latency
expected energy cost
confidence/capability score
availability window
constraints (e.g., “no heavy model loaded” / “charging” / “thermally limited”)
Lease holder selects a winning bid and issues a Work Ticket.
Winning device executes the delegated work strictly within the Work Ticket scope and returns:
result artifact (structured)
minimal explanation
confidence + provenance
Lease holder merges result into chat/task flow.
Why It Exists
Maintains a single execution authority while still leveraging the best available device for compute.
Implications
Delegation is explicit and auditable
No concurrent execution conflicts
Non-lease devices remain non-executable for side effects unless explicitly delegated
Links
Single Executor Guarantee
[Hive Delegated Work Ticket](https://www.notion.so/33ec72bad01381f3b089ec4f59e001ab)
[Hive Bid UI](https://www.notion.so/33ec72bad01381e9b130cb6aac2ce250)

## Related
