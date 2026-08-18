---
title: Retention-Driven Exploration Control
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Retention-Driven Exploration Control.md"]
updated: 2026-07-24
---

# Retention-Driven Exploration Control
Retention-Driven Exploration Control (SA)
Exploration rate adapts based on retention stability within a unit.
Retention is weighted more heavily than short-term comprehension.

Governed by: - ADR-001 - Dual Metric Learning Doctrine - ADR-002 - Authority Separation Doctrine (Alice vs EVE) Parent: - Student/_MOC - Student Engine
Retention Stability Bands (Per Unit)
Unknown: insufficient probes (early unit)
Stable: probes mostly successful; retention buckets trend positive/neutral
Unstable: repeated probe failures or drop/big_drop buckets

Exploration Rates
Unknown: - 70% Preferred (if exists) - 30% Candidate
Stable: - 95% Preferred - 5% Candidate
Unstable: - 75% Preferred - 25% Candidate (or 70/30 for more aggressive search)
If no Preferred exists: - rotate Candidates until one becomes Preferred

Purpose
High retention → consolidate what works
Low retention → broaden search for better methods
Prevents premature locking
Keeps system adaptive year over year

## Related

^[{src_rel}]
