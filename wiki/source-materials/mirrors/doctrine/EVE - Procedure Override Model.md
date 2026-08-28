---
title: "EVE – Procedure Override Model"
type: doctrine
tags: ['lsctech', 'doctrine', 'source-material', 'evo']
updated: 2026-08-24
---

# EVE – Procedure Override Model

## Purpose

Define how school or district administrators can adjust reporting
frequency and granularity without altering:

- core telemetry schema
- privacy safeguards
- identity model
- escalation logic
- statistical neutrality

Procedures modify scheduling and aggregation only.

---

# Default State

By default, EVOlearn reporting aligns with grading periods (~4–5 weeks).

Standard flow:

SA → TA (continuous aggregation)
TA → SE (end-of-cycle cohort summary)
SE → DE (end-of-cycle school summary)

No mid-cycle escalation upward.

---

# What Procedures Can Override

Procedures may adjust:

- reporting frequency (weekly, bi-weekly, custom interval)
- aggregation scope (grade, subject, class, department)
- metric inclusion (retention, efficiency, template usage, etc.)
- anomaly thresholds
- deep-dive triggers
- report formatting (dashboard, PDF, CSV)

Procedures do NOT alter:
- raw telemetry structure
- identity handling
- aggregation thresholds
- consent requirements
- escalation rules

---

# Example Procedure Types

## 1. Increased Frequency

Scope: Grade 7 Math  
Frequency: Weekly  
Metrics: Retention + Template Switch Rate  

---

## 2. Deep Dive Trigger

If retention_probe_success_rate < threshold:
→ request deeper class-level aggregation

---

## 3. Subject Comparison

Aggregate all Grade 8 subjects
→ compare retention variance

---

# Safeguards

All Procedure outputs must:

- respect minimum aggregation threshold (k-anonymity)
- contain no student identifiers
- exclude raw answers or chat content
- remain statistically neutral

If aggregation threshold not met:
→ Procedure result returns “insufficient cohort size”

---

# Execution Model

Procedures are executed by EVE using the internal Task Graph system.

Each Procedure defines:

- task order
- scope filter
- aggregation logic
- scheduling interval
- output artifact type

Procedures may be:

- enabled
- disabled
- reordered
- rescheduled

But definition remains stable unless edited by administrator.

---

# Governance Principle

Procedures provide flexibility without destabilizing:

- identity model
- reporting neutrality
- escalation framework

They adjust timing and scope.
They do not change system philosophy.