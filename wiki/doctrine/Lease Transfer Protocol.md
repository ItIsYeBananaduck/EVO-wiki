---
title: Lease Transfer Protocol
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Lease Transfer Protocol.md"]
updated: 2026-07-24
---

# Lease Transfer Protocol
[Lease Transfer Protocol](https://www.notion.so/33ec72bad013813da225f0ad3eb4ebdb)
Concept
The execution lease transfers when the user switches active devices.
Rule / Mechanism
Lease transfer requires: - explicit user focus change (foreground / active interaction) - revocation of tool grants on previous device - confirmation of lease acquisition on new device
Transfers must be atomic.
Why It Exists
Prevents partial execution across devices.
Implications
Clean handoff
No split-brain execution
Links
[Execution Lease Rule](https://www.notion.so/33ec72bad01381d7846bf9f2cabb72fd)
Scoped Tool Grants

## Related

^[{src_rel}]
