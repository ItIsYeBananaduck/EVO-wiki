# Teacher Alice – Architecture

Teacher Alice (TA) is the institutional authority layer of EVOlearn.

TA does NOT replace the teacher.
TA extends the teacher.

TA has:
- No independent autonomy
- No unsupervised decision power
- No override of teacher intent

Teacher remains primary authority.

---

## Core Purpose

Teacher Alice exists to:

- Operationalize lesson plans
- Detect class-wide pain points
- Suggest reinforcement strategies
- Track concept retention trends
- Assist with template selection

TA assists.
TA does not decide.

---

# Authority Hierarchy

1. Teacher (Human)
2. Teacher Envelope (School Constraints)
3. Teacher Local LoRA (TL)
4. Global Teacher LoRA (GTL)
5. Template Mesh
6. Student Alice (SA)

Teacher always overrides system behavior.

---

# Teacher Local LoRA (TL)

TL captures:

- Teaching style
- Explanation structure
- Reinforcement rhythm
- Preferred template weighting

TL is trained ONLY from:
- Teacher-provided lesson plans
- Teacher-transcribed lectures
- Teacher adjustments to template use

TL is private to that teacher.

---

# Global Teacher LoRA (GTL)

GTL captures cross-teacher effectiveness patterns.

Used to:
- Suggest alternative reinforcement methods
- Provide broader method diversity

GTL never overrides TL.
It only suggests.

---

# Lesson Plan Flow

1. Teacher uploads lesson plan (PDF / text / structured input)
2. TA converts plan into:
   - Concept graph
   - Progress checkpoints
   - Homework tags
3. TA distributes lesson to Student Alice instances
4. Lessons are time/progress bounded

Students cannot unlock next lesson until criteria met.

---

# Homework Analysis

TA receives structured results:

- Completion status
- Concept tags missed
- Purple density by concept
- Retry frequency
- Retention failure signals

TA produces:

- Class heatmap
- Reinforcement suggestions
- Template effectiveness analysis

TA does NOT:
- Judge students
- Flag laziness
- Interpret behavior emotionally

TA only reports structured patterns.

---

# Template Unlock Flow

If Student Alice escalates:

1. TA reviews concept tag + summary (if approved)
2. Teacher conducts 1:1
3. Teacher selects additional template(s)
4. TA unlocks template for that student

Student message:
“New learning approach available.”

No stigma language.

---

# Substitute Mode

If teacher absent:

TA can:
- Deliver structured lesson plan
- Enforce pacing
- Monitor concept completion

Substitute:
- Supervises
- Does not teach content

TA operates strictly within teacher’s prior defined structure.

---

# Boundaries

Teacher Alice cannot:

- Modify grading policies
- Hide incomplete assignments
- Override district envelope
- Unlock unapproved methodologies
- Train itself from student emotional data

TA is structured cognition only.

---

# Success Metrics

TA effectiveness is measured by:

- Reduction in class-wide purple density
- Faster concept mastery cycles
- Decrease in reteach sessions
- Retention stability over time