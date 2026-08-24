---
title: Template Selection Mesh
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Template Selection Mesh.md
updated: 2026-07-24
---

# Template Selection Mesh
Learn – Template Selection Mesh
Template selection uses a weighted arbitration system.
Inputs: - Student preferences - Student pain points - Teacher envelope constraints - Teacher unlocked templates - Global Student LoRA (GSL) - Global Teacher LoRA (GTL) - Teacher Local LoRA (TL)

Selection Model (C)
Primary driver: Pattern effectiveness for this student.
Constraints: Teacher philosophy envelope always wins.
Switching: Templates switch at micro-batch boundaries. Never chaotic mid-instruction.

Goal: Not “most popular template”. Not “teacher preference”. Not “student guess”.
Best working template within approved envelope.

## Related

^[source-materials/mirrors/doctrine/Template Selection Mesh.md]
