# Swarm Merge Rule

## Concept
The lease holder merges Swarm shard outputs into a single coherent response.

## Rule / Mechanism
The lease holder must:
- validate shard outputs against schema
- resolve conflicts deterministically (prefer higher-confidence or more constrained outputs)
- remove duplicates
- ensure final response conforms to architecture atoms and MOCs
- attribute results internally (for audits), not necessarily user-facing unless requested

## Why It Exists
Parallel inference produces multiple partial truths; merging must be controlled.

## Implications
- Prevents contradictory outputs
- Final output stays aligned with your architecture
- Easy to explain “how we got here” if needed

## Links
- [[Method Non-Deviation Rule]]
- [[Whitelisted Instruction Sources]]
- [[Task Audit Log Minimum Fields]]