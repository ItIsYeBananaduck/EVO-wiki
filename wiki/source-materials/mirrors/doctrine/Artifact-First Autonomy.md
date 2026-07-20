# Artifact-First Autonomy

## Concept
Autonomy is powered by persisted artifacts, not by keeping the model resident in memory.

## Rule / Mechanism
All in-session intelligence must write durable outputs (logs, scores, summaries, proposals) that can be consumed later without re-running heavy inference.

## Why It Exists
iOS cannot guarantee that the model remains warm or even resident between sessions.

## Implications
- Nightly inference becomes resumable
- Cold starts remain correct
- Cross-app intelligence becomes practical

## Links
- [[Background Inference Rules]]
- [[Warm State Preservation]]
- [[On-Device First Principle]]