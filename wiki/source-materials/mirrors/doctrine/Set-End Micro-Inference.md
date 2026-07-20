# Set-End Micro-Inference

## Concept
At the end of each set, Alice performs a lightweight inference pass to recommend the next rest period.

## Rule / Mechanism
Set-end inference is limited to rest recommendation only.
It must use current set strain + immediate context and must not trigger heavy reasoning.

## Why It Exists
Rest timing is highly actionable during workouts and benefits from immediate adaptation without destabilizing the plan.

## Implications
- Low latency requirements
- Minimal token budget
- Must be safe if the model is not fully warm

## Links
- [[Live Activity Inference]]
- [[Energy-Aware Inference]]
- [[Non-Intrusive Guidance]]