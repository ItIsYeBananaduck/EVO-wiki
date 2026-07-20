# Effective Weight

## Concept
Effective weight is the actual influence a component has during inference.

## Rule / Mechanism
effective_weight = authority_weight × relevance

If relevance = 0, effective_weight = 0.

## Why It Exists
This prevents blind trust in components that are technically powerful but contextually uninformed.

## Implications
- Cold starts are safe by default
- Personalization grows naturally
- Global rules fade as user confidence increases

## Links
- [[Authority vs Influence]]
- [[Cold Start Safety]]
- [[EVOLoRA Mesh]]