---
title: Exercise-End Intensity Scoring
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Exercise-End Intensity Scoring.md"]
updated: 2026-07-24
---

# Exercise-End Intensity Scoring
[Exercise-End Intensity Scoring](https://www.notion.so/33ec72bad013817eadd0fbd124221931)
Concept
At the end of an exercise (all sets complete), Alice computes an intensity score and appends it to the workout log.
Rule / Mechanism
Intensity scoring is computed once per exercise completion and recorded as a durable log field.
It must not retroactively change completed set data.
Why It Exists
Exercise-level scoring creates stable signals for nightly aggregation and longer-term adaptation.
Implications
Produces lightweight “state artifacts”
Enables trend analysis without heavy inference during the workout
Links
[Background Inference Rules](https://www.notion.so/33ec72bad01381e4861ae0e04fc67d34)
[Artifact-First Autonomy](https://www.notion.so/33ec72bad013819194d5eb34fe70636b)
[User Feedback Interpretation](https://www.notion.so/33ec72bad01381768d12de05f2c0bd11)

## Related
