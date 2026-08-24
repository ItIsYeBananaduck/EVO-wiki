---
title: Error Categorization Sources + Guardrails
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Error Categorization Sources + Guardrails.md
updated: 2026-07-24
---

# Error Categorization Sources + Guardrails
[Error Categorization Sources + Guardrails](https://www.notion.so/33ec72bad013818cb43ad9d96005c863)
Purpose
Define how error categories are assigned while preventing raw student data from entering telemetry or reporting layers.
This document exists to prevent: - raw answer logging - text transcript leakage - behavioral profiling - hidden content retention

Core Rule
Error categorization may produce: - error_category (enum) - error_confidence_bucket (low / medium / high)
It may NOT produce: - raw student answer - reasoning trace - written explanation - essay text - full chat log - math steps
Only enum output is allowed to leave the local categorization layer.

Categorization Sources
1) Deterministic Rule-Based (Preferred)
Used when: - answer format is structured (math, multiple choice, numeric input)
Examples:
incorrect sign → SIGN_ERROR
equation structure mismatch → EQUATION_SETUP_ERROR
unit missing → INSTRUCTIONS_NOT_FOLLOWED
wrong order-of-operations evaluation → ORDER_OF_OPERATIONS
These are safe because: - no language interpretation required - no free text processing - category determined by rule comparison
Rule-based classification is the default whenever possible.

2) Local LLM Classification (Constrained Output)
Used when: - student input is free-form text - reading comprehension explanation - written paragraph response - open-ended conceptual answer
Requirements:
Model runs locally (SA environment)
Prompt strictly constrained to enum selection
Output must be one enum value only
No explanation output allowed
No student text forwarded outside local context
Example allowed output:
error_category: INFERENCE_ERROR error_confidence_bucket: medium
No additional commentary permitted.

Prompt Constraint Template (Internal)
LLM classification prompt must enforce:
“You must return ONLY one value from the allowed enum list. Do not summarize the student’s response. Do not repeat the student’s text. Do not provide explanation.”
System must discard any output beyond: - enum label - optional confidence bucket

Telemetry Boundary
Allowed to propagate upward: - error_category - concept_id - timestamp_bucket - anonymized_student_roll_id - error_confidence_bucket (optional)
Not allowed to propagate: - original response text - micro-lesson transcript - chat conversation - reasoning steps

Error Persistence Policy
Raw student answers: - exist only in active session memory - may be stored temporarily for local re-check - must not enter long-term telemetry logs
Error categories: - may persist within unit window - may contribute to aggregate concept friction metrics - must not persist as permanent student trait

Aggregation Rules
TA may see: - concept-level error category distributions - class-level aggregate counts - error frequency shifts across units
TA may NOT see: - student raw text - student permanent error profiles - template-tier mapping
EVE receives: - only aggregated category distributions - no student roll IDs

Governance Principle
Error categorization is a friction signal, not a psychological evaluation.
Categories describe: “where the learning broke down” not “what kind of student this is.”

## Related

^[source-materials/mirrors/doctrine/Error Categorization Sources + Guardrails.md]
