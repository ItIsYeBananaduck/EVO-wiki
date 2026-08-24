---
title: Artifact-First Autonomy
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Artifact-First Autonomy.md"]
updated: 2026-07-24
---

# Artifact-First Autonomy
Concept
Autonomy is powered by persisted artifacts, not by keeping the model resident in memory.
Rule / Mechanism
All in-session intelligence must write durable outputs (logs, scores, summaries, proposals) that can be consumed later without re-running heavy inference.
Why It Exists
iOS cannot guarantee that the model remains warm or even resident between sessions.
Implications
Nightly inference becomes resumable
Cold starts remain correct
Cross-app intelligence becomes practical
Links
[Background Inference Rules](https://www.notion.so/33ec72bad01381e4861ae0e04fc67d34)
[Warm State Preservation](https://www.notion.so/33ec72bad01381638043ee679eba3da1)
[On-Device First Principle](https://www.notion.so/33ec72bad0138100bbe3ebc5c290f7b8)

## Related
