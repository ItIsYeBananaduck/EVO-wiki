---
title: EVOmind — Signal Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOmind — Signal Model.md"]
updated: 2026-07-24
---

# EVOmind — Signal Model
> NOTE: This is a canonical doctrine note.
> All updates must preserve structure.
> Do not introduce conflicting definitions.

> RULE: All `related` entries must use Obsidian wiki link format.

---

## Purpose
Define how Alice detects, models, and interprets mental and emotional signals using biometric, behavioral, and contextual inputs.

The signal model produces:
- derived mental state signals
- deviation from baseline
- inputs for journaling and adaptation

---

## Core Principle
Signals do not define meaning.

Meaning is learned through:
- baseline comparison
- context
- user feedback

---

## Definitions

**Raw Signals**
Unprocessed inputs such as:
- heart rate (HR)
- heart rate variability (HRV)
- SpO2 (optional)
- typing cadence and latency
- error rate / typos
- tone and structure
- device interaction patterns
- movement (gyro)

---

**Derived Signals (Mind Logs)**
Weighted outputs created from raw signals, representing inferred mental state.

Examples:
- stress level
- emotional intensity
- engagement deviation

---

**Baseline**
A rolling, confidence-weighted model of the user’s normal state.

---

**Context**
Environmental meaning derived from:
- location (what a place represents)
- time
- domain (training, mind, learn, connect)

---

## System Structure

Raw Signals
→ Signal Processing (weighting + normalization)
→ Baseline Comparison
→ Derived Signals (Mind Logs)
→ Context + Comparison Layer
→ Journal Interpretation

---

## Signal Composition

Mind signals are derived from:

### Physiological
- HR
- HRV
- SpO2 (if available)

### Behavioral
- typing speed
- pause duration
- error rate
- interaction frequency

### Expression
- tone
- sentence structure
- word choice

### Environmental
- location meaning
- time of day

### Movement (Critical)
- gyro / activity level
- used to separate physical strain from mental stress

---

## Baseline Model

Baseline is NOT fixed.

It is built from:

### 1. Seed Baseline (Low Confidence)
- onboarding typing samples
- initial interaction patterns

---

### 2. Cross-Domain Signals (Medium Confidence)
- training interactions
- connect usage
- learn engagement

Each tagged by context.

---

### 3. Stable Low-Variance Periods (High Confidence)
- neutral behavior windows
- low signal deviation
- repeated patterns over time

---

### Baseline Properties

- contextual (home, work, gym, etc.)
- dynamic (updates over time)
- confidence-weighted
- never finalized

---

## Deviation Detection

Signals are interpreted as:

- within baseline → neutral
- above baseline → elevated
- below baseline → suppressed

Deviation is relative, not absolute.

---

## Context Integration

Context modifies interpretation but does not override signals.

Examples:

- high HR at gym → likely physical
- high HR at work → likely stress
- high HR at home at night → anomaly

Location is interpreted as:
- what the place means to the user
- not just geographic position

---

## Comparison Layer Integration

The signal model feeds into the comparison layer to classify:

- physical strain (training)
- emotional stress (mind)
- cognitive load (learn)
- mixed states

The signal model does not classify directly.

---

## Role of Conversational Journaling

Journaling provides:

- context for signals
- clarification of meaning
- correction of interpretation

Journaling is required when:
- signals are ambiguous
- deviation is high
- patterns conflict

---

## User Feedback & Correction

User input overrides interpretation.

Sources:

- conversational journaling
- direct correction
- implicit correction through behavior

Corrections:
- update journal interpretation
- influence future signal interpretation
- do not modify raw signals

---

## Anti-Corruption Rules

The system must NOT:

- assume causation from correlation
- treat derived signals as truth
- train on unresolved ambiguity
- override user input with signal inference

---

## Training Signal → Journal Candidate Mapping (EVOtraining)

EVOtraining signals are mapped to `journal_candidate` objects for downstream cognition processing. The mapper emits candidates only — it never writes canonical `journal_entry` records directly and never alters existing training signal producers.

Candidate categories from training signals:

| Category | Description |
|---|---|
| `execution_quality` | Form, phase timing, consistency within a set |
| `fatigue_recovery` | Recovery between sets, HRV patterns |
| `exercise_preference` | Track/exercise correlations (StrainSync) |
| `load_tolerance` | RPE vs weight/reps ratios |
| `adherence_risk` | Plan deviation signals |
| `plan_fit_adjustment` | Persistent misfit between plan and performance |
| `safety_risk_signal` | Extreme load, form break, injury signals |
| `coachability_feedback_pattern` | Repeated live feedback interactions |

See also: [[Journal Entry Schema]], [[EVOtraining — StrainSync System]]

---

## Summary

EVOmind Signal Model:

- derives mental state from multi-source signals
- compares signals to a dynamic baseline
- integrates context and environment
- defers meaning to journaling and user feedback

It enables:

- accurate detection of deviation
- separation of physical, emotional, and cognitive signals
- personalized interpretation over time

## Related

^[{src_rel}]
