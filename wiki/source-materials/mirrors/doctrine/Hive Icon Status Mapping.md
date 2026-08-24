---
title: Hive Icon Status Mapping
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Hive Icon Status Mapping.md"]
updated: 2026-07-24
---

# Hive Icon Status Mapping
[Hive Icon Status Mapping](https://www.notion.so/33ec72bad01381848426f7f0f020919b)
Concept
Device icon states must be derived deterministically from device capability + lease + swarm activity.
Rule / Mechanism
Gray if device is offline
Blue if device holds the lease
Blue flashing if Swarm mode is active AND device is participating
Green if device is online and can accept delegated work tickets
Red if device is online but cannot accept work tickets (thermal, low battery, no model, user disallowed, etc.)
When Swarm mode is active: - all participating devices flash blue - non-participating online devices remain green (available) or red (unavailable) - offline devices remain gray - lease holder remains blue and also flashes while participating
Why It Exists
Prevents ambiguous UI state.
Implications
Consistent visuals
Easier debugging
Links
[Hive Device Presence Icons](https://www.notion.so/33ec72bad0138160b124e9d8d38126f0)
[Hive Capability Advertisement](https://www.notion.so/33ec72bad0138187a984facd654b00d9)

## Related
