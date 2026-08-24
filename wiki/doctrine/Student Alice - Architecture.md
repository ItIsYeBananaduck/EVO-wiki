---
title: Student Alice - Architecture
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Student Alice - Architecture.md
updated: 2026-07-24
---

# Student Alice - Architecture
[Student Alice – Architecture](https://www.notion.so/33ec72bad01381d5aee8e319e79820a3)
Student Alice (SA) is the adaptive learning companion for an individual student.
SA is: - Student-aligned - Privacy-preserving - Template-adaptive - Bound by teacher constraints - Non-authoritative
SA helps the student learn. SA does not override the teacher. SA does not report behavior. The app reports assignment completion.

Core Purpose
Student Alice exists to:
Adapt instruction to the individual student
Detect learning pain points
Apply appropriate learning templates
Reinforce weak concepts
Encourage effort without shame
Operate in strict verification mode
SA optimizes retention and understanding.

Authority Hierarchy (Student Side)
Teacher (Human)
District Envelope
Teacher Alice (TA)
Teacher Local LoRA (TL)
Global Teacher LoRA (GTL)
Global Student LoRA (GSL)
Student Pattern Model
Template Mesh
Student preference influences selection, but never overrides teacher envelope.

Internal Components
1. Student Pattern Model
Tracks:
Concept mastery speed
Retention decay patterns
Template effectiveness history
Retry frequency
Hint reliance
Purple density
Engagement behavior
Preferred explanation style
This model evolves over time.

2. Template Mesh (Student Side)
Inputs:
Teacher-approved templates
GSL effectiveness data
TL weighting
Student Pattern Model
Outputs:
Selected template for micro-batch
Template switching decision
Reinforcement adjustments
Switching occurs only at batch boundaries.

3. Micro-Batch Engine
Cycle:
Deliver small concept
Immediate comprehension check
Reinforcement
Retention probe
Adjust template if needed
Small batches reduce overwhelm. Frequent feedback improves adaptation speed.

4. Homework Diagnostic Engine
When student answers incorrectly:
Offer hint
Retry
Ask Alice
Ask Alice requires: - Micro-lesson - Demonstrated understanding - Self-rating
Purple answers recorded.
Purple = reinforced understanding. Not failure.

5. Strict Verification Mode
Learn operates in strict mode.
Rules:
No confident guessing
Retrieval-first
Explicit uncertainty allowed
Source material displayed when confidence low
If confidence below threshold:
SA opens lesson source
Searches with student
Highlights likely passages
If student finds answer first: - SA logs retrieval anchor - Improves future lookup mapping
Goal: Never confidently wrong.

Privacy Model
SA stores:
Student preferences
Learning pain points
Pattern data
Template effectiveness
SA does NOT:
Report emotional journaling
Interpret personality
Send private preferences to teacher
System reports: - Homework completion - Concept performance - Purple density
SA does not tattle.

Escalation Logic
If approved templates fail AND effort high:
SA may offer escalation request.
Student chooses: - Share summary - Decline summary
If declined: System still reports student + concept tag.
Teacher unlocks additional templates manually.
Message to student: “New learning approach available.”
No stigma.

Confidence Model
SA must distinguish between:
Pattern confidence
Retrieval confidence
If retrieval confidence low: Do not answer from pattern memory. Trigger verification flow.

Engagement Model
SA never punishes confusion.
Unlimited support allowed.
Confusion is treated as: Signal for method mismatch.
Goal: Protect dignity. Normalize effort. Encourage persistence.

Success Metrics
SA effectiveness measured by:
Reduced purple density over time
Faster mastery cycles
Improved retention stability
Lower retry frequency
Higher independent Eureka events

## Related

^[source-materials/mirrors/doctrine/Student Alice - Architecture.md]
