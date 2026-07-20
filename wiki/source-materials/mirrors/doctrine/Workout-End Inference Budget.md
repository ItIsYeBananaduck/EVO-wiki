# Workout-End Inference Budget

## Concept
Workout-end inference may be heavier than set-end inference but must remain bounded.

## Rule / Mechanism
Workout-end summarization has a strict inference budget and must degrade gracefully under energy or thermal constraints.

## Why It Exists
Workout end is common and should never cause slowdowns or battery spikes.

## Implications
- Predictable performance
- Summary quality scales with available resources

## Links
- [[Energy-Aware Inference]]
- [[Inference Budget Ceiling]]
- [[Warm State Preservation]]