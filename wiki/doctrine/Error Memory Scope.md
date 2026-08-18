---
title: Error Memory Scope
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Error Memory Scope.md"]
updated: 2026-07-24
---

# Error Memory Scope
[Error Memory Scope](https://www.notion.so/33ec72bad013813888dfe5f84f4aa0b4)
Error category tracking is scoped to:
(subject_id, unit_id)
Errors do NOT persist as permanent student traits.

Unit Scope Rule
When a unit ends:
error frequency counters reset
stall pattern history resets
template error correlations reset
Only aggregate statistics (anonymized, bucketed) may persist upward.

Why Unit-Scoped?
Prevents student labeling
Avoids cross-topic contamination
Reflects reality: mistakes vary by concept
Encourages fresh evaluation each unit
Aligns with retention-weighted adaptation model

What Can Persist (Limited)
Across units within the same subject, SA may retain:
General retention stability trends
Exploration bias adjustments
Template prior weights (as Candidate, not Preferred)
But NOT:
Specific error categories
Error frequency counters
Student error “profiles”

Governance Principle
Errors describe friction within a concept. They do not describe the student.

## Related

^[{src_rel}]
