# Procedure Scheduling Model

## Purpose

Define how EVE executes Procedures over time.

Procedures control:
- frequency
- scope
- grouping
- threshold checks
- artifact generation

They do not alter telemetry schema or privacy safeguards.

---

# Scheduling Types

## 1. Fixed Interval

- Weekly
- Bi-weekly
- Monthly
- Grading Period (~4–5 weeks)

Example:
Run Grade 7 Math retention report every Friday at 4pm.

---

## 2. Event-Based

Triggered when:
- grading period closes
- threshold breach occurs
- administrator manually triggers run

Example:
If retention_probe_success_rate < threshold
→ trigger deep-dive procedure

---

## 3. Manual

Administrator initiates:
- one-time run
- custom scope selection

---

# Execution Flow

1. Scheduler activates Procedure.
2. EVE pulls eligible aggregated reports.
3. Task Graph executes in defined order.
4. Output artifact generated.
5. Results stored locally on institutional server.
6. No external transmission.

---

# Constraints

- Must respect aggregation minimum threshold.
- Must not request raw student-level data.
- Must not override consent model.
- Must not access chat content.

---

# Scheduling Safeguards

- Procedures may be:
  - enabled
  - disabled
  - rescheduled
- Logs retained for audit trail.
- No retroactive metric recalculation unless explicitly configured.

---

# Governance Principle

Scheduling changes timing only.
It does not change:
- metric definitions
- identity protections
- statistical neutrality