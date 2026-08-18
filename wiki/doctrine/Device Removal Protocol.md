---
title: Device Removal Protocol
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Device Removal Protocol.md"]
updated: 2026-07-24
---

# Device Removal Protocol
[Device Removal Protocol](https://www.notion.so/33ec72bad01381af827be2473a294c78)
Concept
Users can remove Hive devices from settings safely.
Rule / Mechanism
Only the current main device may remove devices. Removal: - revokes trust credentials immediately - blocks the removed device from reading hive state - requires re-pairing to rejoin
Why It Exists
Device removal is a security action and must be centrally enforced.
Implications
Lost/stolen devices can be cut off
Hive remains secure
Links
[Main Device Root of Trust](https://www.notion.so/33ec72bad0138174b8caee1ba7b83dfb)
[Hive Security Settings Maintenance Mode](https://www.notion.so/33ec72bad01381f5acf3ef74f28e8fe8)

## Related

^[{src_rel}]
