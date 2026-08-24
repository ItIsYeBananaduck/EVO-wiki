---
title: Schema - Progress Report (Level 0)
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Schema - Progress Report (Level 0).md
updated: 2026-07-24
---

# Schema - Progress Report (Level 0)
[Schema – Progress Report (Level 0)](https://www.notion.so/33ec72bad0138149958bc4cd9a1460a6)
Purpose
A device-level structured report emitted by a Student App (Learn) or Worker App (Enterprise).
This is the lowest level of the pipeline. Everything above is aggregation only.

Envelope
{ “type”: “PROGRESS_REPORT”, “domain”: “learn | enterprise”, “schema_version”: 1, “generated_at”: “ISO-8601”, “scope”: { “org_unit”: “class_id | department_id | team_id”, “grade”: “optional (Learn)”, “subject”: “optional (Learn)”, “unit_id”: “optional (Learn)” }, “privacy”: { “identity_mode”: “anonymous | role-mapped”, “k_anonymity_ready”: true }, “metrics”: { … } }

Metrics (Learn)
“metrics”: { “progress”: { “checkpoint_completion_rate”: number, “concept_mastery_rate”: number }, “retention”: { “retention_probe_success_rate”: number, “decay_risk_index”: number }, “efficiency”: { “time_to_mastery_estimate”: number, “attempts_per_concept_avg”: number }, “support_intensity”: { “hint_count”: number, “retry_count”: number, “ask_alice_count”: number, “purple_count”: number, “eureka_count”: number }, “template_usage”: { “templates_used”: [ { “template_id”: string, “batches”: number } ], “switch_count”: number }, “strict_mode”: { “source_open_events”: number, “unknown_resolved_via_source_rate”: number } }
No raw answers. No text. No chat.

Metrics (Enterprise)
“metrics”: { “progress”: { … }, “efficiency”: { … }, “quality”: { … }, “variance”: { … } }
Enterprise schema is domain-pack-defined.

## Related

^[source-materials/mirrors/doctrine/Schema - Progress Report (Level 0).md]
