---
title: Procedure Scheduling Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Procedure Scheduling Model.md"]
updated: 2026-07-24
---

# Procedure Scheduling Model
[Procedure Scheduling Model](https://www.notion.so/33ec72bad013818c8580cfde7855a94a)
Purpose
Define how EVE executes Procedures over time.
Procedures control: - frequency - scope - grouping - threshold checks - artifact generation
They do not alter telemetry schema or privacy safeguards.

Scheduling Types
1. Fixed Interval
Weekly
Bi-weekly
Monthly
Grading Period (~4–5 weeks)
Example: Run Grade 7 Math retention report every Friday at 4pm.

2. Event-Based
Triggered when: - grading period closes - threshold breach occurs - administrator manually triggers run
Example: If retention_probe_success_rate < threshold → trigger deep-dive procedure

3. Manual
Administrator initiates: - one-time run - custom scope selection

Execution Flow
Scheduler activates Procedure.
EVE pulls eligible aggregated reports.
Task Graph executes in defined order.
Output artifact generated.
Results stored locally on institutional server.
No external transmission.

Constraints
Must respect aggregation minimum threshold.
Must not request raw student-level data.
Must not override consent model.
Must not access chat content.

Scheduling Safeguards
Procedures may be:
enabled
disabled
rescheduled
Logs retained for audit trail.
No retroactive metric recalculation unless explicitly configured.

Governance Principle
Scheduling changes timing only. It does not change: - metric definitions - identity protections - statistical neutrality

---

## Related

^[{src_rel}]
