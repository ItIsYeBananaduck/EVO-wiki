# Hive Bid Protocol

## Concept
If the lease holder cannot (or should not) run an inference/task, it may offer the work to the rest of the Hive to bid.

## Rule / Mechanism
1) Lease holder emits a Bid Request containing:
- task_id / prompt_id
- required work type (plan / summarize / heavy inference / tool-mapping)
- estimated token budget
- urgency (interactive vs background-acceptable)
- required constraints (no tools, tools, secrets, etc.)

2) Hive members respond with bids describing:
- expected latency
- expected energy cost
- confidence/capability score
- availability window
- constraints (e.g., "no heavy model loaded" / "charging" / "thermally limited")

3) Lease holder selects a winning bid and issues a Work Ticket.

4) Winning device executes the delegated work strictly within the Work Ticket scope and returns:
- result artifact (structured)
- minimal explanation
- confidence + provenance

5) Lease holder merges result into chat/task flow.

## Why It Exists
Maintains a single execution authority while still leveraging the best available device for compute.

## Implications
- Delegation is explicit and auditable
- No concurrent execution conflicts
- Non-lease devices remain non-executable for side effects unless explicitly delegated

## Links
- [[Single Executor Guarantee]]
- [[Hive Delegated Work Ticket]]
- [[Hive Bid UI]]