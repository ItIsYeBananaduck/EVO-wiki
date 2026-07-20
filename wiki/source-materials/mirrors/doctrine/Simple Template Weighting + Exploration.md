# Simple Template Weighting + Exploration (SA)

Goal:
Provide a simple, explainable template selection system that adapts by context.

---

## Tiers (Per Context)

Context = subject + unit_id (or concept cluster)

Each template is one of:
- Preferred
- Candidate
- Avoid

No global learning-type labels.

---

## Selection Rules

Default:
1. Try Preferred
2. If stall → try Candidate
3. If stall persists → rotate Candidates
4. If Candidates exhausted → offer escalation (consent required)

---

## Win/Loss Tracking

SA tracks per template per context:
- wins (small int)
- losses (small int)

Win examples:
- retention probe improves
- time-to-mastery improves
- support intensity decreases
- student answers independently after lesson

Loss examples:
- repeated stall persists
- retention probe fails after use
- support intensity spikes

---

## Tier Movement Thresholds

- Candidate → Preferred:
  - 3 wins and ≤1 loss (recent window)

- Preferred → Candidate:
  - 2 losses within window

- Candidate → Avoid:
  - 3 losses within window

- Avoid → Candidate:
  - teacher unlock OR exploration test yields 2 wins

---

## Exploration Rates

Early-year (first N weeks):
- 70% Preferred
- 30% Candidate

Normal:
- 90% Preferred
- 10% Candidate

If no Preferred exists:
- rotate Candidates until one becomes Preferred.

---

## Yearly Warm Start

At new academic year start:
- last year’s Preferred templates initialize as Candidate
- regain Preferred after 2 early-year wins in same context

Rationale:
- year/grade/context changes may alter effectiveness
- warm start without permanent assumptions