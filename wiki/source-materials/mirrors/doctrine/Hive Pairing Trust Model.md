---
title: Hive Pairing Trust Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Hive Pairing Trust Model.md"]
updated: 2026-07-24
---

# Hive Pairing Trust Model
[Hive Pairing Trust Model](https://www.notion.so/33ec72bad0138194b7eade86eb4d05cf)
Concept
Hive devices must trust each other through explicit pairing.
Rule / Mechanism
Pairing establishes: - device identity - encryption keys for sync transport - authorization to access shared state and LoRAs
Unpaired devices cannot: - read hive chat - read tasks - receive work tickets - access synced LoRAs/logs
Why It Exists
Multi-device sync is a major security boundary.
Implications
Pairing is required for any participation
Devices can be revoked from the Hive
Links
[Device Linking via QR](https://www.notion.so/33ec72bad01381f3b125d481c91a1f66)
[Secret Isolation Rule](https://www.notion.so/33ec72bad013813e9eb6ead8af1141ad)

## Related
