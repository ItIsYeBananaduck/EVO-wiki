# Hive Bid Scoring Rule

## Concept
Bid selection must be deterministic and safety-aware.

## Rule / Mechanism
Default bid scoring prioritizes:
1) Ability to complete within constraints (hard gate)
2) Lowest latency for interactive work OR lowest energy for background work
3) Highest confidence score
4) Best thermal/battery posture (prefer charging + cool)
5) Lowest disruption (prefer idle devices)

Lease holder may auto-select the top score unless user overrides.

## Why It Exists
Avoids unpredictable delegation choices and protects user experience.

## Implications
- Consistent outcomes
- Prevents heavy work on weak knowing devices
- Easy to tune without rewriting logic

## Links
- [[Hive Capability Advertisement]]
- [[Hive Bid UI]]