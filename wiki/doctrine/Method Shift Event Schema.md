---
title: Method Shift Event Schema
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Method Shift Event Schema.md"]
updated: 2026-07-24
---

# Method Shift Event Schema (Bucketed)
Purpose: Report the impact of SA internal method shifts without exposing: - template identity - tier state - student labeling signals
Only coarse performance deltas are transmitted upward.

Event Scope
method_shift_event {
subject_id: string unit_id: string
timestamp_bucket: string # e.g., “2026-W07” or “UnitWeek-2” anonymized_student_roll_id: string # rolling identifier
shift_trigger_type: enum { stall, retention_drop, support_spike, exploration_test }
# Bucketed Deltas (coarse) retention_delta_bucket: enum { big_drop, drop, neutral, gain, big_gain }
time_to_mastery_delta_bucket: enum { much_slower, slower, neutral, faster, much_faster }
support_intensity_delta_bucket: enum { much_harder, harder, neutral, easier, much_easier }
shift_effectiveness_bucket: enum { improved, neutral, worsened }
}

## Related

^[{src_rel}]
