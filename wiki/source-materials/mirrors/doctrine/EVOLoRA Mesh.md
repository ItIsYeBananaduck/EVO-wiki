
# EVOLoRA Mesh

## Concept
The EVOLoRA Mesh is a dynamic decision system that blends multiple LoRA adapters using authority and relevance rather than static weights.

## Rule / Mechanism
Each LoRA contributes to inference based on its effective weight.

effective_weight = authority_weight × relevance

LoRAs with zero relevance have zero influence, regardless of authority.

## Why It Exists
Static LoRA blending cannot safely support personalization, cold starts, or long-term adaptation.

The Mesh allows:
- Safe early behavior
- Gradual personalization
- Conflict resolution without hard rules

## Implications
- Personalization is earned over time
- Safety dominates early sessions
- Different apps can reuse the same intelligence model

## Links
- [[Authority vs Influence]]
- [[Effective Weight]]
- [[User LoRA Silence Condition]]
- [[Cold Start Safety]]