---
title: Student Alice - Core Architecture
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Student Alice - Core Architecture.md"]
updated: 2026-07-24
---

# Student Alice - Core Architecture
[Student Alice – Kids School Mode](https://www.notion.so/33ec72bad01381c6bea3d2f7cbe47b11)
This profile applies when EVOlearn is used under a school/teacher environment.

Authority Layer (Kids)
District Envelope (philosophy + constraints)
Teacher (Human)
Teacher Alice (TA) supervisory layer
Teacher Local LoRA (TL)
Global Teacher LoRA (GTL)
Global Student LoRA (GSL)
Learner Pattern Model
Template Mesh
Teacher envelope always wins.

Ingestion
Lesson plans distributed from TA → SA
Homework imported/assigned by teacher/school
Student does not need to provide materials manually

Reporting
System reports assignment completion and results
Purple answers visible to teacher dashboards as pain point signal
SA does not “tattle” — reporting is infrastructure level

Escalation
If templates fail and effort is high: - SA offers student a choice: - share structured summary with teacher - or decline summary - if declined, system still sends student + concept tag Teacher may unlock additional templates for that student.
Student-facing language: “New approach available.” No stigma framing.

## Related

^[{src_rel}]
