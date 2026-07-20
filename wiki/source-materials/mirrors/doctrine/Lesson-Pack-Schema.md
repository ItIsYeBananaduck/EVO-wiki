# Lesson Pack Schema (Teacher Alice → Student Alice)

## Design Goals
- Stable contract across all teachers
- Supports unique teaching styles without breaking data model
- Kids mode is curriculum-bounded by default
- Enables offline caching and versioning

---

## Top-Level Object: LessonPack

### Required
- pack_id (string)
- version (int)
- class_id (string)
- teacher_id (string)
- title (string)
- grade_band (enum)
- subject (enum)
- objectives (string[])
- prerequisites (string[])
- unlock_rules (UnlockRules)
- sections (Section[])
- assessments (Assessment[])
- method_profile (MethodProfile)
- safety_constraints (SafetyConstraints)

### Optional
- schedule (ScheduleHints)
- resources (Resource[])
- teacher_notes_for_students (string[])
- teacher_notes_private (string[])  // never exported to students
- localization (locale tags)

---

## Section

### Required
- section_id
- title
- concept_tags (string[])
- content_blocks (ContentBlock[])
- required (bool)
- mastery_rule (MasteryRule)

### Optional
- differentiation (DifferentiationOptions)
- pacing_hint (enum: fast|normal|slow)

---

## ContentBlock

### Required
- block_id
- block_type (enum: explain|example|guided_practice|independent_practice|reflection|story)
- reading_level (enum: below|at|above)
- core_text (string)

### Optional
- visuals (Resource[])
- audio (Resource[])
- interactive_template (GameTemplateRef)   // template-driven only
- hint_bank (Hint[])
- vocab (VocabTerm[])
- worked_steps (string[])
- teacher_voice_notes (string[])  // short, controlled snippets

---

## Assessment

### Required
- assessment_id
- type (enum: quiz|worksheet|oral_check|project_checkpoint)
- items (AssessmentItem[])
- passing_rule (PassingRule)
- retake_policy (RetakePolicy)

---

## AssessmentItem

### Required
- item_id
- prompt
- format (enum: mcq|short_answer|fill_blank|matching|ordering)
- correct_answer (string or structured)
- concept_tags (string[])

### Optional
- distractors (string[])
- rubric (string[])
- hint (string)

---

## UnlockRules

### Modes
- time_based (unlock_after_date)
- mastery_based (threshold)
- hybrid (earliest_date + mastery_threshold)

### Required Fields
- mode (enum: time|mastery|hybrid)
- lesson_completion (LessonCompletionCriteria)

---

## LessonCompletionCriteria
- required_sections (string[])      // section_ids
- minimum_mastery (0..1)            // trend-based score, not perfection
- minimum_attempts (int)
- allow_teacher_override (bool)

---

## MasteryRule
- metric (enum: trend_success|help_reduction|spaced_recall)
- threshold (0..1)
- window (enum: last_3|last_5|last_10)
- no_penalty_for_mistakes (bool=true)

---

## MethodProfile (Teacher "How I Teach")

### Required
- method_id
- structure (enum: I_DO_WE_DO_YOU_DO | Socratic | Story_First | Practice_First | Mixed)
- tone (enum: calm|encouraging|energetic) // Kids shell still bounded
- explanation_preferences (string[])      // "use analogies", "use visuals", etc.
- pacing_style (enum: micro_chunks|standard|deep_dive)
- feedback_style (enum: gentle|direct|question_led)

### Optional
- common_misconceptions (Misconception[])
- preferred_examples (string[])
- banned_phrases (string[])
- reward_style (enum: quiet|light|none)
- accessibility (AccessibilityPrefs)

---

## SafetyConstraints

### Required
- curriculum_bounded (bool=true for Kids)
- allowed_topics (string[])
- disallowed_topics (string[])
- generation_limits:
  - allow_open_chat (bool=false for Kids)
  - allow_external_knowledge (bool=false for Kids)
  - allow_game_generation (bool=false) // only templates
  - max_variants_per_block (int)