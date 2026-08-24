---
title: TA-SA - Communication Protocol
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/TA-SA - Communication Protocol.md"]
updated: 2026-07-24
---

# TA-SA - Communication Protocol
TA ↔ SA Communication Protocol
Teacher Alice (TA) and Student Alice (SA) do not communicate conversationally.
They exchange structured, bounded messages.
Protocol is:
Event-driven
Stateless per message
Versioned
Auditable
Policy-aware

Design Principles
No freeform LLM-to-LLM dialogue
No emotional content exchange
No student preference leakage unless explicitly approved
All communication passes through System Layer
TA cannot access SA internal pattern model directly

Message Types
1. Lesson Distribution
TA → SA
{ type: “LESSON_PLAN”, lesson_id: string, concept_graph: […], checkpoint_structure: […], homework_tags: […], allowed_templates: […], time_bounds: {…}, version: number }
Purpose: Deliver structured lesson to student.
SA Response: ACK or ERROR (version mismatch, policy conflict, etc.)

2. Homework Result Report
SA → TA
{ type: “HOMEWORK_RESULT”, student_id: string, lesson_id: string, completion_status: boolean, concept_performance: [ { concept_tag, correct_rate, retry_count, purple_count } ], retention_flags: […], timestamp }
Purpose: Provide structured performance summary.
No emotional data. No journaling content. No private preferences.

3. Escalation Request
SA → TA (via System)
{ type: “ESCALATION_REQUEST”, student_id: string, concept_tag: string, effort_score: number, template_attempts: […], summary: optional (if student approved) }
If student declines summary: Only student_id + concept_tag sent.
TA cannot force summary exposure.

4. Template Unlock
TA → SA
{ type: “TEMPLATE_UNLOCK”, student_id: string, concept_tag: string, additional_templates: […], expiry: optional, reason_code: optional }
SA merges new allowed templates into mesh.
Switching still occurs only at batch boundary.

5. Class Heatmap Request
TA → System
{ type: “CLASS_ANALYSIS_REQUEST”, lesson_id: string }
System aggregates: - purple density - retry frequency - concept mastery variance
TA receives: Aggregated metrics only.
No raw pattern models exposed.

Forbidden Communication
TA cannot: - Query SA internal confidence scores - Access student preference notes - Access private self-study ingestion content - Modify student pattern model directly
SA cannot: - Override teacher envelope - Hide incomplete assignments - Send emotional interpretations

Versioning
Each message includes:
{ protocol_version: number, schema_version: number }
This allows backward compatibility during updates.

Failure Handling
If mismatch detected:
Reject message
Log event
Notify System
No silent failure
Determinism > flexibility

## Related
