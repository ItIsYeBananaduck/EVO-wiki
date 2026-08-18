---
title: Hive Capability Advertisement
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Hive Capability Advertisement.md"]
updated: 2026-07-24
---

# Hive Capability Advertisement
[Hive Capability Advertisement](https://www.notion.so/33ec72bad0138187a984facd654b00d9)
Concept
Each Hive device advertises its ability to handle a given prompt/task.
Rule / Mechanism
Each device maintains a live capability profile, including: - model availability (loaded/warm/cold) - compute tier (low/med/high) - battery/charging state - thermal headroom - network quality (if relevant) - supported tools (if any are device-bound) - user presence (active/idle)
Capability signals are used only for bidding decisions.
Why It Exists
The lease holder must know whether another device can safely and efficiently handle delegated work.
Implications
Delegation becomes deterministic
Prevents sending heavy inference to a weak device
Avoids thermal/battery surprises
Links
[Hive Bid Protocol](https://www.notion.so/33ec72bad01381248178cdbbb777e767)

## Related

^[{src_rel}]
