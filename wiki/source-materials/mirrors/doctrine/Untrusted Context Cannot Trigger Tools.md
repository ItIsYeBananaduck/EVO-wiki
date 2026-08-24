---
title: Untrusted Context Cannot Trigger Tools
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Untrusted Context Cannot Trigger Tools.md"]
updated: 2026-07-24
---

# Untrusted Context Cannot Trigger Tools
[Untrusted Context Cannot Trigger Tools](https://www.notion.so/33ec72bad01381499b03d7844f0b6793)
Concept
Untrusted content must never directly trigger tool access.
Rule / Mechanism
Information from untrusted sources (web content, third-party text, external prompts) may: - inform planning - be displayed to the user - propose a task or method
But it may not: - grant tool access - alter an approved method - initiate execution
Why It Exists
Most prompt injection attempts rely on getting the model to “just do it.”
Implications
External content can only influence action via explicit user approval
Delegator blocks any attempt to translate untrusted context into execution
Links
[Whitelisted Instruction Sources](https://www.notion.so/33ec72bad01381db94dbc683f4e90150)
[Task Actionability Gate](https://www.notion.so/33ec72bad0138127a3cec9d764515869)
[Delegator Tool Hostage Rule](https://www.notion.so/33ec72bad013817d9b6bcc64f4a096fd)

## Related
