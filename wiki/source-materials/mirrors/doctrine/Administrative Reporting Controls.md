---
title: Administrative Reporting Controls
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Administrative Reporting Controls.md"]
updated: 2026-07-24
---

# Administrative Reporting Controls
[Administrative Reporting Controls](https://www.notion.so/33ec72bad0138108b116dd30e3ecfeaf)
Purpose
Define what administrators can configure within EVOlearn reporting.
Administrators control: - report frequency - aggregation scope - included metrics - threshold alerts - artifact format
They do not control: - identity mapping - raw telemetry schema - escalation logic - consent requirements

Control Categories
1. Scope Controls
by grade
by subject
by class cluster
by department

2. Metric Inclusion
Selectable metrics: - retention rate - efficiency metrics - template usage - concept hotspot frequency - purple density - variance analysis

3. Threshold Settings
Admins may define: - retention floor % - acceptable variance band - template switch spike threshold
Thresholds trigger: - deeper aggregation - anomaly flag - manual review recommendation

4. Output Controls
dashboard
PDF summary
CSV export
heatmap visualization
All output remains local to institution.

Guardrails
Administrators cannot: - access student identity - drill into single-student metrics - override aggregation threshold - view chat content - alter consent-based escalation behavior

[Philosophy](https://www.notion.so/33ec72bad0138193a0bbce9f89d79395)
Administrators control reporting structure. They do not control individual student data visibility.

## Related
