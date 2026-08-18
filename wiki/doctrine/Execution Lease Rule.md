---
title: Execution Lease Rule
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Execution Lease Rule.md"]
updated: 2026-07-24
---

# Execution Lease Rule
[Execution Lease Rule](https://www.notion.so/33ec72bad01381d7846bf9f2cabb72fd)
Concept
Only one Hive member may execute tasks at any given time.
Rule / Mechanism
The execution lease: - is held by the device the user is actively using - grants the ability to execute authorized tasks - must be released before another device can execute
Devices without the lease may not execute tools.
Why It Exists
Prevents race conditions, duplicate actions, and conflicting side effects.
Implications
Safe device switching
Deterministic execution
Clear responsibility
Links
[Delegator Tool Hostage Rule](https://www.notion.so/33ec72bad013817d9b6bcc64f4a096fd)
[Single Alice Illusion](https://www.notion.so/33ec72bad01381ea8395c1c9c35f9066)

## Related

^[{src_rel}]
