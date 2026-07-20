# Schema – Cohort Summary (Level 1)

## Purpose
TA/MA aggregates device-level reports into a cohort-level summary.

Cohort = class section (Learn) or department/team (Enterprise).

---

{
  "type": "COHORT_SUMMARY",
  "domain": "learn | enterprise",
  "schema_version": 1,
  "generated_at": "ISO-8601",
  "scope": {
    "org_unit": "class_id | department_id",
    "grade": "optional",
    "subject": "optional",
    "unit_id": "optional"
  },
  "privacy": {
    "student_identity": "none",
    "aggregation_min_count": number
  },
  "distributions": {
    "checkpoint_completion_rate": { "p25": n, "p50": n, "p75": n },
    "retention_probe_success_rate": { "p25": n, "p50": n, "p75": n },
    "purple_density": { "p25": n, "p50": n, "p75": n },
    "template_switch_rate": { "p25": n, "p50": n, "p75": n }
  },
  "signals": {
    "concept_hotspots": [
      { "concept_tag": string, "severity": number }
    ],
    "unit_risk": number
  }
}