# Per-Unit Adaptation Window

Template adaptation in SA is scoped to:

(student, subject, unit_id)

---

## Reset Behavior

When a new unit begins:

- win/loss counters reset
- tier status recalculates
- all templates start as Candidate
- previous unit Preferred becomes High-Priority Candidate

---

## Exploration

Early in unit:
- 70% Preferred (if exists)
- 30% Candidate

If no Preferred:
- rotate Candidates until one earns promotion

---

## Stall Triggers (Unit Scoped)

Examples:
- 2 failed attempts after hint
- repeated purple conversions
- retention probe decline

Triggers:
- rotate template
- re-evaluate weighting

---

## End-of-Unit

- retention probe
- archive stats
- update yearly carry-over memory