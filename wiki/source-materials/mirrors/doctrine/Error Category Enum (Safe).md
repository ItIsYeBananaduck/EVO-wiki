# Error Category Enum (Safe)

## Purpose

Classify common mistake types using coarse categories to support:

- stall detection
- targeted micro-lessons
- retention probe interpretation
- method shift triggers
- teacher-facing aggregate pain points
- EVE concept cluster analysis

This enum must NOT store:
- raw student answers
- full work steps
- personal text explanations
- chat logs

Only category + concept tag.

---

# Representation

error_event {
  subject_id: string
  unit_id: string
  concept_id: string          # or concept_tag
  timestamp_bucket: string
  anonymized_student_roll_id: string

  error_category: enum
  error_confidence_bucket: enum { low, medium, high }
}

Confidence bucket is internal to SA; only aggregated distributions propagate upward.

---

# Error Categories (v1)

## General / Cross-Subject

- NONE (correct / no error)
- MISREAD_PROMPT
- ATTENTION_SLIP              # accidental click / skipped step / rushed
- CARELESS_ARITHMETIC         # simple arithmetic slip unrelated to concept mastery
- TIME_PRESSURE               # rushed due to time constraints
- INSTRUCTIONS_NOT_FOLLOWED    # format requirement missed (show work, units, etc.)
- VOCAB_DEFINITION_GAP         # doesn't know term used in prompt
- CONTEXT_CONFUSION            # doesn't know what the question is asking overall

---

## Math / Quantitative

- SIGN_ERROR                  # +/- mistake
- ORDER_OF_OPERATIONS
- DISTRIBUTIVE_PROPERTY
- COMBINE_LIKE_TERMS
- FRACTION_MANIPULATION
- DECIMAL_PLACE_VALUE
- VARIABLE_ISOLATION
- EQUATION_SETUP_ERROR         # wrong equation from word problem
- UNIT_CONVERSION
- GEOMETRY_VISUALIZATION_GAP   # diagram / spatial reasoning mismatch
- GRAPH_INTERPRETATION_ERROR
- FORMULA_SELECTION_ERROR      # chose wrong formula
- ALGEBRAIC_REARRANGEMENT_ERROR

---

## Reading / Writing / Language Arts

- MAIN_IDEA_MISIDENTIFIED
- INFERENCE_ERROR
- EVIDENCE_SELECTION_WEAK
- SEQUENCING_CONFUSION
- VOCAB_IN_CONTEXT_ERROR
- GRAMMAR_SYNTAX_ERROR
- STRUCTURE_ORGANIZATION_WEAK
- TONE_PURPOSE_CONFUSION

---

## Science (Conceptual)

- CONCEPT_MISCONCEPTION        # incorrect mental model
- CAUSE_EFFECT_CONFUSION
- VARIABLE_CONTROL_ERROR        # experiment design misunderstanding
- DATA_TABLE_MISREAD
- GRAPH_INTERPRETATION_ERROR    # shared category allowed
- UNIT_CONVERSION               # shared category allowed

---

## History / Social Studies

- TIMELINE_CONFUSION
- CAUSE_EFFECT_CONFUSION        # shared category allowed
- SOURCE_INTERPRETATION_ERROR
- CONTEXT_GAP                   # missing background context
- COMPARISON_CONTRAST_WEAK

---

# Stall Detection Integration

Stall risk increases when:

- same error_category repeats within unit window
- across multiple attempts after hint
- or repeats across different problems under same concept tag

This supports:
- targeted micro lessons
- template switching
- escalation readiness

---

# Privacy Safeguards

- Store only enum + concept tag.
- Never store raw answers in telemetry layer.
- No identity mapping beyond rolling ID.
- Only aggregated category distributions are visible to TA/EVE.

---

# Governance Principle

Error categories represent "what kind of friction occurred",
not "how smart the student is".

They are used to improve method selection and scaffolding.