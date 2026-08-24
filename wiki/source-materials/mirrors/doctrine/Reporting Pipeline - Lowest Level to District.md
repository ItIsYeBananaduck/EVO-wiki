---
title: Reporting Pipeline - Lowest Level to District
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Reporting Pipeline - Lowest Level to District.md"]
updated: 2026-07-24
---

# Reporting Pipeline - Lowest Level to District
Reporting Pipeline – Lowest Level to District (Learn + Enterprise)
Principle
Define the lowest-level report format first. Higher levels only aggregate and reformat.
No raw content. No chat/journaling. No freeform LLM-to-LLM dialogue.

Level 0 – App Report (Student App / Worker App)
Sender: - Student device (Learn) - Worker device (Enterprise)
Receiver: - TA (Teacher AI) in Learn - MA (Manager AI) in Enterprise
Report type: PROGRESS_REPORT
Allowed fields (examples): - progress metrics - efficiency metrics - retention metrics (Learn only) - support intensity metrics (purple/hints/retries)
No identity required for students. For workers, identity policy is org-defined (usually role-based).

Level 1 – Manager Aggregation (TA / MA)
TA/MA actions: - aggregate multiple app reports into cohort summaries - produce heatmaps / distributions - detect local anomalies (class/department) - forward aggregated metrics upward
TA/MA cannot: - forward raw user content - forward identifiable student telemetry - perform surveillance ranking
Output type: COHORT_SUMMARY

Level 2 – School EVE (SE)
SE ingests: - cohort summaries from TA (Learn) - cohort summaries from MA (Enterprise)
SE produces: - school-wide aggregates - grade/subject/unit aggregates (Learn) - department-level aggregates (Enterprise)
SE forwards upward: SCHOOL_SUMMARY (aggregated, grouped)

Level 3 – District EVE (DE)
DE ingests: - school summaries from SE
DE produces: - district-wide aggregates - grouped by grade (Learn) and school - multi-school comparisons (distribution-based)
DE does not receive: - student identifiers - raw class-level telemetry (unless explicitly configured and still anonymized)
Output type: DISTRICT_SUMMARY

## Related
