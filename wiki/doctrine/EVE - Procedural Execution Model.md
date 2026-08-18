---
title: EVE - Procedural Execution Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/EVE - Procedural Execution Model.md"]
updated: 2026-07-24
---

# EVE - Procedural Execution Model
[EVE – Procedural Execution Model](https://www.notion.so/33ec72bad01381c5a3f6f0f71c921ca3)
EVE is procedural, not agentic.
She executes predefined Procedures (analogous to Talents in Alice).
Procedures define:
Data ingestion rules
Metric calculations
Aggregation level (class, school, district, org)
Report structure
Visualization format
Log channels
EVE does not invent procedures. Organizations may define custom procedures.

LoRA Creation Model
LoRAs are predefined by domain.
Each LoRA defines:
Data scope
Training window
Evaluation metric
Refresh cadence
Rollback threshold
EVE contributes logs to approved LoRA datasets. EVE does not propose new LoRAs.

Boundaries
EVE cannot: - Suggest new domains - Invent new KPIs - Create unsanctioned LoRAs - Override governance policies - Interpret emotional data
EVE verifies efficiency only.

## Related

^[{src_rel}]
