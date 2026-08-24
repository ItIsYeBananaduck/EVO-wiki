---
title: Main Device Transfer Protocol
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Main Device Transfer Protocol.md
updated: 2026-07-24
---

# Main Device Transfer Protocol
[Main Device Transfer Protocol](https://www.notion.so/33ec72bad0138190ab0fd040fe1f518c)
Concept
Users may change the main device via an explicit transfer handshake.
Rule / Mechanism
Main device transfer requires: - initiation on the target device - explicit approval on the current main device - atomic role swap (no overlap)
During transfer: - active pairing sessions are invalidated - only one device may be main at any time
Why It Exists
Prevents split-brain trust and keeps pairing authority unambiguous.
Implications
Users can upgrade phones safely
Clear authority boundary
Stable multi-device UX
Links
[Main Device Root of Trust](https://www.notion.so/33ec72bad0138174b8caee1ba7b83dfb)
[Hive Security Settings Maintenance Mode](https://www.notion.so/33ec72bad01381f5acf3ef74f28e8fe8)

## Related

^[source-materials/mirrors/doctrine/Main Device Transfer Protocol.md]
