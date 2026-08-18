---
title: ADR-001 - Dual Metric Learning Doctrine
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/ADR-001 - Dual Metric Learning Doctrine.md"]
updated: 2026-07-24
---

# ADR-001 - Dual Metric Learning Doctrine
[ADR-001 — Dual Metric Learning Doctrine](https://www.notion.so/33ec72bad01381879188c703b5b11826)
Status: AcceptedDate: YYYY-MM-DDOwner: EVOlearn Core

Context
Traditional education systems evaluate students primarily through compliance-based comprehension metrics.
However, comprehension in the moment does not guarantee long-term retention.
EVOlearn aims to:
Improve knowledge durability
Preserve psychological safety
Remain institution-compatible
Avoid labeling students as incapable
A structural separation between comprehension and retention is required.

Decision
EVOlearn will operate on a Dual-Metric Model:
Compliance (School-Facing)
Retention Stability (Student-Facing)
These metrics are independent but related.
Compliance ensures institutional compatibility.Retention ensures durable learning.
Neither metric overrides the other globally.

Compliance Definition
Compliance measures:
Ability to demonstrate required methodology
Proper formatting
Alignment with curriculum standards
Compliance is: - Required for school mode - Never optional in homework submission

Retention Definition
Retention Stability measures:
Durability of knowledge over time
Time-weighted recall accuracy
Reinforcement resilience
Pattern-based stabilization
Retention bands: - Building - Strengthening - Stable
Retention is:
Non-judgmental
Fluctuation-normalized
Pattern-driven (not single-event driven)

Authority Boundaries
Alice: - User-aligned - Optimizes retention and comprehension - Cannot override institutional pacing
EVE: - Institution-aligned - Receives aggregated performance deltas - Cannot identify individual students

Rationale
This doctrine prevents:
Identity damage from conflating formatting with intelligence
Institutional conflict
Overcorrection from noisy single-event signals
Shame-based learning mechanics
It enables:
Durable knowledge
Parent reassurance
Transparent divergence handling
Ethical adaptive learning

Consequences
Positive: - Improved long-term retention modeling - Reduced student labeling - Cleaner institutional integration - Clear architectural separation
Trade-offs: - Increased system complexity - More nuanced UI communication - Need for careful language design

Invariants
The following may not be altered without formal revision:
Retention and compliance remain separate metrics.
Retention bands must not imply intelligence.
Institutional mode cannot allow pacing override.
Stability shifts are pattern-based, not event-based.
No cross-student retention comparison.

Revision Policy
Any future proposal to: - Merge metrics - Rank students publicly - Remove fluctuation normalization - Override institutional alignment
Requires new ADR and governance review.

## Related

^[{src_rel}]
