---
title: Schema - Cohort Summary (Level 1)
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Schema - Cohort Summary (Level 1).md"]
updated: 2026-07-24
---

# Schema - Cohort Summary (Level 1)
[Schema – Cohort Summary (Level 1)](https://www.notion.so/33ec72bad0138142aa25de4cf6f14d25)
Purpose
TA/MA aggregates device-level reports into a cohort-level summary.
Cohort = class section (Learn) or department/team (Enterprise).

{ “type”: “COHORT_SUMMARY”, “domain”: “learn | enterprise”, “schema_version”: 1, “generated_at”: “ISO-8601”, “scope”: { “org_unit”: “class_id | department_id”, “grade”: “optional”, “subject”: “optional”, “unit_id”: “optional” }, “privacy”: { “student_identity”: “none”, “aggregation_min_count”: number }, “distributions”: { “checkpoint_completion_rate”: { “p25”: n, “p50”: n, “p75”: n }, “retention_probe_success_rate”: { “p25”: n, “p50”: n, “p75”: n }, “purple_density”: { “p25”: n, “p50”: n, “p75”: n }, “template_switch_rate”: { “p25”: n, “p50”: n, “p75”: n } }, “signals”: { “concept_hotspots”: [ { “concept_tag”: string, “severity”: number } ], “unit_risk”: number } }

## Related
