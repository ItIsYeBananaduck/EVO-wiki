---
title: Hive Shared State Backbone
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Hive Shared State Backbone.md"]
updated: 2026-07-24
---

# Hive Shared State Backbone
[Hive Shared State Backbone](https://www.notion.so/33ec72bad01381a2845acb86cd36ef48)
Concept
All Hive devices share a single source of truth for Alice state.
Rule / Mechanism
Hive must synchronize: - chat history - task manager state - approved methods library - Talents (metadata + snapshots) - LoRA artifacts and versions - logs/artifacts needed for continuity
Why It Exists
Without shared state, “one Alice” breaks across devices.
Implications
State sync is mandatory
Conflicts must resolve deterministically
Devices can come and go without breaking continuity
Links
[Single Alice Illusion](https://www.notion.so/33ec72bad01381ea8395c1c9c35f9066)
[LoRA Artifact Sync](https://www.notion.so/33ec72bad0138155b304efaefc565899)
[Hive Log Sync](https://www.notion.so/33ec72bad01381afb3cedbc0c5e34719)

## Related

^[{src_rel}]
