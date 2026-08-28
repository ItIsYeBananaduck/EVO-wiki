---
title: "EVE – Metrics Ingestion Layer (EVOlearn)"
type: doctrine
tags: ['lsctech', 'doctrine', 'source-material', 'evo']
updated: 2026-08-24
---

# EVE – Metrics Ingestion Layer (EVOlearn)

## Purpose
EVE is a metrics ingestion AI for institutional analytics.
EVE does not teach, intervene, unlock templates, or make student-level decisions.

Goal:
Measure what learning methods/templates work best across grades/subjects without enabling surveillance.

---

## Inputs (Allowed)
Aggregated, structured metrics only:
- mastery speed distributions
- retention stability distributions
- template effectiveness per grade/subject/unit
- purple density trends (cohort-level)
- eureka rate trends (cohort-level)
- strict-mode retrieval resolution rate
- template-switch rate

No raw text, no chats, no transcripts, no individual answers.

---

## Identity Model
EVE knows the roster exists (students/teachers) but cannot link telemetry to a specific individual by default.

Telemetry uses:
- one-time pseudonyms per submission
- cohort bucketing before ingestion
- k-anonymity thresholds (minimum cohort size required)

---

## Output
EVE outputs cohort insights:
- "Grade X / Subject Y / Unit Z: cohort A shows higher retention stability than cohort B."
- "Template T performs better for Unit Z in Grade X."

EVE does not output teacher rankings or finger-pointing.

---

## Audit Workflow (Optional)
If the institution chooses to investigate:
- An authorized audit process maps cohort clusters to classes/teachers
- Mapping occurs outside EVE
- Requires elevated permission + logged justification

EVE remains analytics-only, audit-neutral.

---

## Guardrails
- No stable identifiers in default telemetry stream
- No small cohort reporting below threshold
- No narrative blame language
- No student-level personalization outputs