---
title: EVE - Procedures as Task Graphs
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/EVE - Procedures as Task Graphs.md"]
updated: 2026-07-24
---

# EVE - Procedures as Task Graphs
[EVE – Procedures as Task Graphs](https://www.notion.so/33ec72bad01381108201ee9969406a0c)
Procedures are analytics-only workflows.
They contain Task Graphs: - ordered tasks (or DAG if needed) - each task is strictly data processing (no external actions)
Procedures are inherently safer than Talents because: - no agency - no side effects - output-only behavior

Admin Controls
Admins can: - enable/disable a procedure - enable/disable tasks within the procedure - reorder tasks - assign tasks to runners (nodes) - schedule execution frequency
Admins do NOT need to “approve actions” because procedures have no actions.

Stability Model
Procedure definition remains stable. Operational behavior is controlled by: - on/off toggles - task inclusion/exclusion - task order - schedule
This provides: - determinism - repeatability - low governance overhead

## Related
