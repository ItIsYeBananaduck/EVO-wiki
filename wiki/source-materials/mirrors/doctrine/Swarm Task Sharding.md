# Swarm Task Sharding

## Concept
Heavy inference is decomposed into bounded inference sub-tasks.

## Rule / Mechanism
The lease holder must shard work into tasks with:
- clear objective
- explicit input slice (chat range, relevant artifacts)
- output schema (structured result)
- token/time budget
- disallowed actions (no tools, no secrets unless explicitly permitted)

Examples of shard types:
- extract requirements
- generate method draft
- risk analysis
- summarization
- alternative solution generation
- contradiction detection
- UI copy drafting
- acceptance criteria drafting

## Why It Exists
Parallel work must be bounded and composable.

## Implications
- Shards are auditable and reproducible
- Results can be merged deterministically

## Links
- [[Swarm Parallel Inference]]
- [[Swarm Merge Rule]]