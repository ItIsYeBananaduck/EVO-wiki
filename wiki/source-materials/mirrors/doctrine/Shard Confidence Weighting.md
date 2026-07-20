# Shard Confidence Weighting

## Concept
When merging Swarm shard outputs, confidence signals influence precedence.

## Rule / Mechanism
Each shard result may include:
- confidence score (0–1)
- self-reported uncertainty
- device provenance

Merge precedence rules:
1) Schema-valid outputs always outrank invalid ones
2) Higher-confidence outputs preferred when conflicts arise
3) Outputs from shards with narrower scope preferred over broader ones
4) Architecture consistency violations override confidence (hard veto)

Confidence is advisory, not authoritative.

## Why It Exists
Parallel inference often yields overlapping or conflicting outputs.

## Implications
- Deterministic merges
- Reduced hallucinated dominance
- Architecture rules always win

## Links
- [[Swarm Merge Rule]]
- [[Architecture Consistency Check Shard]]