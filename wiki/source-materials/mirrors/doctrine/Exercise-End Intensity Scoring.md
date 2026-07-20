# Exercise-End Intensity Scoring

## Concept
At the end of an exercise (all sets complete), Alice computes an intensity score and appends it to the workout log.

## Rule / Mechanism
Intensity scoring is computed once per exercise completion and recorded as a durable log field.

It must not retroactively change completed set data.

## Why It Exists
Exercise-level scoring creates stable signals for nightly aggregation and longer-term adaptation.

## Implications
- Produces lightweight “state artifacts”
- Enables trend analysis without heavy inference during the workout

## Links
- [[Background Inference Rules]]
- [[Artifact-First Autonomy]]
- [[User Feedback Interpretation]]