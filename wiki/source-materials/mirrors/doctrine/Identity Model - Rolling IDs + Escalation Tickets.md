# Identity Model – Rolling IDs + Escalation Tickets

Goal:
Allow human intervention without creating persistent identity linkage.

No mapping key exists.

---

## Channel A: Anonymous Telemetry

Used for:
- progress metrics
- retention metrics
- efficiency metrics
- cohort distributions

Properties:
- no names
- no stable identifiers
- rolling IDs rotate on report boundaries
- batched submission
- optional jitter to prevent timing correlation

Telemetry may be aggregated by TA/SE/DE/EVE but never linked to a student identity.

---

## Channel B: Escalation Tickets (Human Help)

Used for:
- template escalation requests
- teacher intervention needs

Properties:
- may include student name (required for human follow-up)
- contains concept tag and optional student-approved summary
- does NOT include telemetry IDs
- is NOT forwarded to EVE analytics pipelines
- exists for action by teacher only

---

## Rolling ID Rule

After escalation, next telemetry report uses a new rolling ID.
This prevents persistent linkage between:
- named escalation events
- anonymous performance telemetry

---

## Anti-Reidentification Safeguards

- batch + jitter telemetry submission
- minimum cohort threshold for upward reporting
- no small-cohort drilldowns