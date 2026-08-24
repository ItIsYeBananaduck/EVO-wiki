---
title: Device Linking via QR
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Device Linking via QR.md"]
updated: 2026-07-24
---

# Device Linking via QR
[Device Linking via QR](https://www.notion.so/33ec72bad01381f3b125d481c91a1f66)
Concept
Devices join a user’s Hive by scanning a QR code (WhatsApp-style pairing).
Rule / Mechanism
Pairing flow: 1) Primary device displays QR code containing a short-lived pairing payload 2) New device scans QR to request to join the Hive 3) Devices establish trust and exchange keys 4) New device begins initial sync (state + LoRAs as needed)
Pairing payload must be time-limited and single-use.
Why It Exists
Fast, user-friendly linking without account friction.
Implications
Secure onboarding to the Hive
Prevents accidental device joins
Enables multi-device continuity
Links
[Hive Pairing Trust Model](https://www.notion.so/33ec72bad0138194b7eade86eb4d05cf)
[Hive Shared State Backbone](https://www.notion.so/33ec72bad01381a2845acb86cd36ef48)

## Related
