---
title: Hive Bid Scoring Rule
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Hive Bid Scoring Rule.md"]
updated: 2026-07-24
---

# Hive Bid Scoring Rule
[Hive Bid Scoring Rule](https://www.notion.so/33ec72bad01381fda085e3d571b8a6ff)
Concept
Bid selection must be deterministic and safety-aware.
Rule / Mechanism
Default bid scoring prioritizes: 1) Ability to complete within constraints (hard gate) 2) Lowest latency for interactive work OR lowest energy for background work 3) Highest confidence score 4) Best thermal/battery posture (prefer charging + cool) 5) Lowest disruption (prefer idle devices)
Lease holder may auto-select the top score unless user overrides.
Why It Exists
Avoids unpredictable delegation choices and protects user experience.
Implications
Consistent outcomes
Prevents heavy work on weak knowing devices
Easy to tune without rewriting logic
Links
[Hive Capability Advertisement](https://www.notion.so/33ec72bad0138187a984facd654b00d9)
[Hive Bid UI](https://www.notion.so/33ec72bad01381e9b130cb6aac2ce250)

## Related
