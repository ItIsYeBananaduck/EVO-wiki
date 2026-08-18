---
title: Hive Device Presence Icons
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Hive Device Presence Icons.md"]
updated: 2026-07-24
---

# Hive Device Presence Icons
[Hive Device Presence Icons](https://www.notion.so/33ec72bad0138160b124e9d8d38126f0)
Concept
Hive and Swarm status is represented to the user via small glowing device icons.
Rule / Mechanism
Each device in the Hive is represented by a user-configurable icon (with defaults per device type). Icon colors reflect status:
Green: device ON + available
Red: device ON + unavailable (constraints prevent participation)
Blue: device is the current lease holder (active executor)
Blue flashing: Swarm is active (multiple devices running inference in parallel)
Gray: device OFF
Icons must be visible in relevant views (chat/task context) without disrupting the user.
Why It Exists
Transparency builds trust and makes distributed compute feel legible.
Implications
Users understand where work is happening
Users can diagnose “why it’s slow”
Supports user mental model of the Hive/Swarm
Links
[Execution Lease Rule](https://www.notion.so/33ec72bad01381d7846bf9f2cabb72fd)
[Swarm Parallel Inference](https://www.notion.so/33ec72bad013810eaa8ac7df235f10d0)

## Related

^[{src_rel}]
