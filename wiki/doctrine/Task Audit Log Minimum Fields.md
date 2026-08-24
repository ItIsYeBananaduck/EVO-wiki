---
title: Task Audit Log Minimum Fields
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Task Audit Log Minimum Fields.md
updated: 2026-07-24
---

# Task Audit Log Minimum Fields
[Task Audit Log Minimum Fields](https://www.notion.so/33ec72bad01381aa9d87d0a77aa0cada)
Concept
Task history must be reviewable with enough detail to understand what happened.
Rule / Mechanism
For each executed task, the system must retain at minimum: - Task ID, title, timestamps - Authorization path (approved method vs Talent) - Method or Talent reference - Tool call log summary (what tools were used, not necessarily full payloads) - Outcome (completed/failed/canceled) - User-visible output artifacts
Why It Exists
Without minimum fields, history becomes meaningless and trust erodes.
Implications
Consistent “what happened” UI
Easier investigation of unexpected behavior
Enables safe promotion decisions
Links
[Task Transparency Retention](https://www.notion.so/33ec72bad013811ca9abdbbde305dc48)
Scoped Tool Grants
[Delegator Tool Hostage Rule](https://www.notion.so/33ec72bad013817d9b6bcc64f4a096fd)

---

Related notes: [[Task Transparency Retention]], [[Task Actionability Gate]]

## Related

^[source-materials/mirrors/doctrine/Task Audit Log Minimum Fields.md]
