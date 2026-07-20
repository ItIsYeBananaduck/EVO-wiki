# Swarm Activation Criteria

## Concept
Swarm mode is triggered only when local single-node compute is insufficient or undesirable.

## Rule / Mechanism
Swarm may be activated if:
- estimated inference cost exceeds the lease holder budget, AND
- no single device can complete within constraints, OR
- parallel execution is significantly safer (thermal/battery) than single-node.

Swarm must not activate for trivial prompts.

## Why It Exists
Swarm is powerful but expensive; it should be reserved for heavy work.

## Implications
- Predictable energy use
- Reduced unnecessary complexity

## Links
- [[Inference Budget Ceiling]]
- [[Energy-Aware Inference]]
- [[Swarm Parallel Inference]]