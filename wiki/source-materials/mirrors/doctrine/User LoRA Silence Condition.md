# User LoRA Silence Condition

## Concept
User-specific intelligence must not act without sufficient experience.

## Rule / Mechanism
If the User LoRA has no relevant signal, it contributes zero influence.

This applies even if authority weight is high.

## Why It Exists
False personalization is more dangerous than generic guidance.

## Implications
- Global Trainer dominates early
- Users are never penalized for being new
- Adaptation feels intentional and trustworthy

## Links
- [[Effective Weight]]
- [[Cold Start Safety]]
- [[EVOLoRA Mesh]]