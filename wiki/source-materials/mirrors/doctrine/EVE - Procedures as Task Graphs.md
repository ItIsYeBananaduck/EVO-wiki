# EVE – Procedures as Task Graphs

Procedures are analytics-only workflows.

They contain Task Graphs:
- ordered tasks (or DAG if needed)
- each task is strictly data processing (no external actions)

Procedures are inherently safer than Talents because:
- no agency
- no side effects
- output-only behavior

---

## Admin Controls

Admins can:
- enable/disable a procedure
- enable/disable tasks within the procedure
- reorder tasks
- assign tasks to runners (nodes)
- schedule execution frequency

Admins do NOT need to "approve actions" because procedures have no actions.

---

## Stability Model

Procedure definition remains stable.
Operational behavior is controlled by:
- on/off toggles
- task inclusion/exclusion
- task order
- schedule

This provides:
- determinism
- repeatability
- low governance overhead