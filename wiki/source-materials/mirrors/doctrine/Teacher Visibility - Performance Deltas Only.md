# Teacher Visibility – Performance Deltas Only

SA maintains template tier states privately (on-device).

TA does not receive:
- template IDs
- template tier labels (Preferred/Candidate/Avoid)
- student method preferences

TA receives only:
- aggregated performance deltas
- distribution summaries
- method-shift counts (no labels)

---

## Allowed Delta Signals (Examples)

- retention_delta (pre vs post)
- time_to_mastery_delta
- support_intensity_delta
- method_shift_count
- shift_effectiveness_bucket (improved/neutral/worse)

---

## Rationale

Prevents:
- student labeling
- template-based tracking
- teacher ranking dynamics
- privacy erosion

Preserves:
- institutional trust
- student dignity
- neutral reporting