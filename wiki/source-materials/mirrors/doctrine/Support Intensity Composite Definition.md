# Support Intensity Composite Definition (Learn)

## Purpose

Support Intensity estimates how much assistance a student required
to complete a learning interaction.

It is used for:
- stall detection (unit + subject scoped)
- method shift triggers
- template effectiveness scoring
- reporting deltas (bucketed)

It is NOT used for grading or punishment.

---

# Core Concept

Support Intensity is a weighted composite score calculated from
interaction events.

The score is evaluated per (subject_id, unit_id) and over a short window.

---

# Components (v1)

## 1) Attempts
- independent_attempt_count

Rationale:
More attempts can be normal, but repeated attempts with no progress
suggest higher support need.

Weight: +1 per additional attempt after first

---

## 2) Hint Usage
- hint_used_count

Weight: +2 per hint

---

## 3) Retry Loop Count
- retry_count_after_hint

Weight: +2 per retry after hint

---

## 4) Micro Lesson Dependence
- micro_lesson_count
- micro_lesson_duration_bucket (short / medium / long)

Weights:
- micro_lesson_count: +3 each
- duration bucket:
  - short: +0
  - medium: +1
  - long: +2

---

## 5) "Ask Alice" Completion (Purple)
- purple_answer_count

Purple means:
Student tried → hint → retried → completed micro lesson → demonstrated understanding → Alice provided answer.

Weight: +4 each purple answer

Note:
Purple is not "bad"; it represents extra effort + guided resolution.
It is simply a stronger support intensity signal.

---

## 6) Eureka Exits
- eureka_exit_count

Eureka means the student ended the micro lesson early because they
understood and answered independently.

Weight: -2 each (reduces intensity)

---

# Composite Score (Simple)

support_intensity_score =
  (attempts_weighted)
+ (hints_weighted)
+ (retries_weighted)
+ (micro_lesson_weighted)
+ (purple_weighted)
- (eureka_weighted)

All fields are event counts or coarse buckets.
No raw content required.

---

# Normalization (Unit Window)

To avoid day-to-day noise, compute:

- per-session intensity score
- then smooth to a unit-window average

Unit window = (subject_id, unit_id)

---

# Bucketing (for reporting deltas)

When comparing two windows (pre vs post method shift),
compute percentage change in average intensity score and bucket it:

- much_harder: ≥ +25%
- harder: +10% to +24%
- neutral: -9% to +9%
- easier: -10% to -24%
- much_easier: ≤ -25%

---

# Governance Notes

- Not used for punishment or grading.
- Not shared as a student-specific label.
- Only aggregated distributions propagate upward (TA/SE/DE/EVE).
- Used to detect friction and guide adaptive support.

---

# Design Philosophy

High support intensity is not failure.
It is information:
- the method may not match the student/context
- the concept may require different scaffolding
- the student may need a break or spaced retry