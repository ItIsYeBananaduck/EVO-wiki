---
title: Connect Hive Architecture
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Connect Hive Architecture.md
updated: 2026-07-24
---

# Connect Hive Architecture
Hive = coordination layer.

Responsibilities:

- Node discovery (WebRTC P2P)
- Fallback relay (Cloudflare if required)
- Capability detection
- Task distribution
- Encrypted internal communication

Rules:

- User never sees model-to-model chat
- One leader node at a time
- Nodes can enter and exit safely
- Leader can be reassigned

Principle:

Hive ≠ Swarm. Hive coordinates. Swarm computes.

## Related

^[source-materials/mirrors/doctrine/Connect Hive Architecture.md]
