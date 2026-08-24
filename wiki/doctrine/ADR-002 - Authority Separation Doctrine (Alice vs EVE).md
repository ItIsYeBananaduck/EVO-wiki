---
title: ADR-002 - Authority Separation Doctrine (Alice vs EVE)
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/ADR-002 - Authority Separation Doctrine (Alice vs EVE).md
updated: 2026-07-24
---

# ADR-002 - Authority Separation Doctrine (Alice vs EVE)
[ADR-002 — Authority Separation Doctrine (Alice vs EVE)](https://www.notion.so/33ec72bad0138138addaff60b77e479e)
Status: AcceptedDate: YYYY-MM-DDOwner: EVOlearn Core

Context
EVOlearn introduces two AI roles:
Alice (Student/Parent-facing intelligence)
EVE (Institution-facing aggregation intelligence)
Without strict boundaries, adaptive systems risk:
Identity tracking
Behavioral inference
Institutional overreach
Loss of user trust
Centralized data control
The system must preserve:
Student dignity
Privacy sovereignty
Institutional reporting capability
Ethical separation of powers

Decision
EVOlearn will enforce strict authority separation:
Alice = User-AlignedEVE = Institution-Aligned
These roles must remain structurally and operationally distinct.

I. Alice Doctrine
Alice is aligned to:
The student (primary)
The parent (in tutoring mode)
The learner’s cognitive growth
Alice is NOT aligned to:
Institutional grading enforcement
Attendance policing
Behavioral interpretation
Psychological diagnosis
Alice may:
Optimize retention
Adapt templates
Recommend reinforcement
Provide transparency about curriculum alignment
Alice may NOT:
Report student identity to EVE
Flag behavioral concerns institutionally
Rank students against peers
Infer motives from absence
Alice is a regulation and optimization tool — not an authority.

II. EVE Doctrine
EVE is aligned to:
Institutional efficiency
Aggregated retention performance
Template effectiveness analysis
Grade-level and school-level deltas
EVE may:
Aggregate anonymized performance signals
Identify template performance trends
Generate administrative reports
Process procedure-based analytics
EVE may NOT:
Identify individual students
Map retention history across reports
Request raw student-level cognitive data
Infer identity from aggregated data
No mapping keys are stored on central servers.
EVE operates locally within school or district infrastructure.

III. Data Flow Rules
Student App → TA → SE → DE
Data moving upward must be:
Aggregated
Bucketed
Identity-rotated
Non-reversible
No global student identifier exists.
Identity rotation occurs between reporting cycles.
Escalation chats remain isolated from reporting pipeline.

IV. Escalation Boundary
When deeper template analysis is required:
Student approval required
Teacher notified via secure channel
Escalation chat isolated
Reporting ID rolls independently
Escalation data does not propagate upward automatically.

V. Institutional Scope Limits
In school mode:
Retention scoped to academic year
No cross-grade identity continuity
No longitudinal institutional labeling
In adult mode:
Longitudinal modeling permitted
User-owned data
No institutional visibility

Rationale
This separation:
Prevents surveillance creep
Protects student dignity
Preserves institutional usability
Enables ethical adaptive learning
Maintains trust architecture
Without separation, the system risks becoming:
Performance policing
Predictive labeling
Behavioral scoring infrastructure
This is explicitly prohibited.

Invariants
The following may not be altered without new ADR:
Alice cannot report student identity to EVE.
EVE cannot reconstruct identity from reports.
No central mapping key exists.
Identity rotation is mandatory between report cycles.
Escalation chats are siloed from analytics.

Trade-offs
Reduced granular institutional insight
Increased engineering complexity
Requires strict reporting discipline
These trade-offs are accepted in favor of:
Privacy sovereignty
Ethical architecture
Long-term trust

## Related

^[source-materials/mirrors/doctrine/ADR-002 - Authority Separation Doctrine (Alice vs EVE).md]
