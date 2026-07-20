# Energy-Aware Inference

## Concept
Inference must respect device power and thermal constraints.

## Rule / Mechanism
Inference intensity adapts based on:
- Battery level
- Thermal state
- Execution context (foreground vs background)

Non-essential inference is deferred under constraint.

## Why It Exists
An intelligent system that drains power is not intelligent.

## Implications
- Reduced energy impact
- Longer live sessions
- Predictable performance

## Links
- [[On-Device First Principle]]
- [[Live Activity Inference]]