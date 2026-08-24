---
title: Method Approval Path
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Method Approval Path.md"]
updated: 2026-07-24
---

# Method Approval Path
## Concept

User approval of a Method authorizes task execution for that specific run.

---

## Rule / Mechanism

Alice must present the task’s Method to the user, including:

- steps
- tools required
- expected outputs
- risks or side effects (if applicable)

Upon explicit user approval:

- the task becomes executable
- tool access is granted in a scoped manner

---

## Reuse Rule

Reusing a Method does not inherit approval.

Each execution still requires explicit approval unless the execution is happening through a valid Talent.

This means:

- an approved Method may be reused structurally
- but it is not independently executable without fresh approval
- only Talents remove the need for per-task re-approval

---

## Why It Exists

User intent must remain explicit before governed execution occurs.

---

## Implications

- approval is task-scoped unless promoted
- approval can be revoked
- approved Methods may later qualify for Talent promotion
- reuse of a Method does not itself create autonomy

---

## Related Notes

- [Task Actionability Gate](https://www.notion.so/33ec72bad0138127a3cec9d764515869)
- Scoped Tool Grants
- [Talent Promotion Rule](https://www.notion.so/33ec72bad013814389d2efd20e39c2c6)
- [Approved Method Library](https://www.notion.so/33ec72bad01381029fe3fca456cd9372)
- [EVOconnect Method Specification Model](https://www.notion.so/33dc72bad01381969e43e43864cb35ef)
