---
title: Teacher Alice - Architecture
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Teacher Alice - Architecture.md
updated: 2026-07-24
---

# Teacher Alice - Architecture
[Teacher Alice – Architecture](https://www.notion.so/33ec72bad0138135a863d7210dd723d7)
Teacher Alice (TA) is the institutional authority layer of EVOlearn.
TA does NOT replace the teacher. TA extends the teacher.
TA has: - No independent autonomy - No unsupervised decision power - No override of teacher intent
Teacher remains primary authority.

Core Purpose
Teacher Alice exists to:
Operationalize lesson plans
Detect class-wide pain points
Suggest reinforcement strategies
Track concept retention trends
Assist with template selection
TA assists. TA does not decide.

Authority Hierarchy
Teacher (Human)
Teacher Envelope (School Constraints)
Teacher Local LoRA (TL)
Global Teacher LoRA (GTL)
Template Mesh
Student Alice (SA)
Teacher always overrides system behavior.

Teacher Local LoRA (TL)
TL captures:
Teaching style
Explanation structure
Reinforcement rhythm
Preferred template weighting
TL is trained ONLY from: - Teacher-provided lesson plans - Teacher-transcribed lectures - Teacher adjustments to template use
TL is private to that teacher.

Global Teacher LoRA (GTL)
GTL captures cross-teacher effectiveness patterns.
Used to: - Suggest alternative reinforcement methods - Provide broader method diversity
GTL never overrides TL. It only suggests.

Lesson Plan Flow
Teacher uploads lesson plan (PDF / text / structured input)
TA converts plan into:
Concept graph
Progress checkpoints
Homework tags
TA distributes lesson to Student Alice instances
Lessons are time/progress bounded
Students cannot unlock next lesson until criteria met.

Homework Analysis
TA receives structured results:
Completion status
Concept tags missed
Purple density by concept
Retry frequency
Retention failure signals
TA produces:
Class heatmap
Reinforcement suggestions
Template effectiveness analysis
TA does NOT: - Judge students - Flag laziness - Interpret behavior emotionally
TA only reports structured patterns.

Template Unlock Flow
If Student Alice escalates:
TA reviews concept tag + summary (if approved)
Teacher conducts 1:1
Teacher selects additional template(s)
TA unlocks template for that student
Student message: “New learning approach available.”
No stigma language.

Substitute Mode
If teacher absent:
TA can: - Deliver structured lesson plan - Enforce pacing - Monitor concept completion
Substitute: - Supervises - Does not teach content
TA operates strictly within teacher’s prior defined structure.

Boundaries
Teacher Alice cannot:
Modify grading policies
Hide incomplete assignments
Override district envelope
Unlock unapproved methodologies
Train itself from student emotional data
TA is structured cognition only.

Success Metrics
TA effectiveness is measured by:
Reduction in class-wide purple density
Faster concept mastery cycles
Decrease in reteach sessions
Retention stability over time

## Related

^[source-materials/mirrors/doctrine/Teacher Alice - Architecture.md]
