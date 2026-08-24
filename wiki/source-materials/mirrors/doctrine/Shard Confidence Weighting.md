---
title: Shard Confidence Weighting
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Shard Confidence Weighting.md"]
updated: 2026-07-24
---

# Shard Confidence Weighting
[Shard Confidence Weighting](https://www.notion.so/33ec72bad01381f8ae0dfc693899d392)
Concept
When merging Swarm shard outputs, confidence signals influence precedence.
Rule / Mechanism
Each shard result may include: - confidence score (0–1) - self-reported uncertainty - device provenance
Merge precedence rules: 1) Schema-valid outputs always outrank invalid ones 2) Higher-confidence outputs preferred when conflicts arise 3) Outputs from shards with narrower scope preferred over broader ones 4) Architecture consistency violations override confidence (hard veto)
Confidence is advisory, not authoritative.
Why It Exists
Parallel inference often yields overlapping or conflicting outputs.
Implications
Deterministic merges
Reduced hallucinated dominance
Architecture rules always win
Links
[Swarm Merge Rule](https://www.notion.so/33ec72bad0138162ba34d5e3dcc576f1)
Architecture Consistency Check Shard

## Related
