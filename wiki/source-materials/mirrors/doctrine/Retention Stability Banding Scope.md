# Retention Stability Banding Scope

Retention stability bands are computed at:

(subject_id, unit_id)

Reason:
- retention patterns vary by subject
- units represent concept clusters
- prevents cross-topic contamination
- keeps adaptation explainable and auditable

---

## Band Values

- Unknown
- Stable
- Unstable

Bands influence:
- exploration rate
- template selection bias
- tier movement weighting (retention-heavy)

---

## Governance

Bands are internal to SA.
Only aggregated bucket distributions propagate upward.
No subject/unit stability label is exposed to TA or EVE at student level.