---
title: Set-End Micro-Inference
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Set-End Micro-Inference.md"]
updated: 2026-07-24
---

# Set-End Micro-Inference
[Set-End Micro-Inference](https://www.notion.so/33ec72bad01381499c43d002520faafb)
Concept
At the end of each set, Alice performs a lightweight inference pass to recommend the next rest period.
Rule / Mechanism
Set-end inference is limited to rest recommendation only. It must use current set strain + immediate context and must not trigger heavy reasoning.
Why It Exists
Rest timing is highly actionable during workouts and benefits from immediate adaptation without destabilizing the plan.
Implications
Low latency requirements
Minimal token budget
Must be safe if the model is not fully warm
Links
[Live Activity Inference](https://www.notion.so/33ec72bad01381c78f2ff2ab2e8dab34)
[Energy-Aware Inference](https://www.notion.so/33ec72bad013818aa564f83af05fb2ba)
[Non-Intrusive Guidance](https://www.notion.so/33ec72bad013810e9792e13c03de6618)

---

Related notes: [[Micro-Batch Cycle]]

## Related
