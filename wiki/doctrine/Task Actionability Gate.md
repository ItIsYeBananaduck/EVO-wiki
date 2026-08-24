---
title: Task Actionability Gate
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Task Actionability Gate.md
updated: 2026-07-24
---

# Task Actionability Gate
## Concept

Tasks are non-actionable by default. Execution requires an explicit authorization path.

## Rule / Mechanism

A task becomes actionable only if at least one authorization path is satisfied:

### Method Approval Path

- Alice presents the Method to the user
- the user explicitly approves the Method for that task

### Talent Path

- the task references a valid Talent
- the Talent is within scope for that task

If neither path is satisfied:

- the task remains non-actionable
- Delegator blocks execution
- tool access is denied

- Prevents silent autonomy and ensures execution is governed.
Implications
- Planning may occur without execution
- Execution must be explainable and auditable
- Tools are never accessible by default

### Links

- [Talent Definition](https://www.notion.so/33ec72bad0138124922ee770d3aebbc0)
- [Method Approval Path](https://www.notion.so/33ec72bad01381fa9b3ec4729d474082)
- [Delegator Tool Hostage Rule](https://www.notion.so/33ec72bad013817d9b6bcc64f4a096fd)

---

Related notes: [[Task Audit Log Minimum Fields]]

##

## Related

^[source-materials/mirrors/doctrine/Task Actionability Gate.md]
