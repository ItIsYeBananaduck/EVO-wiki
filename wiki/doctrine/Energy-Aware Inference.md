---
title: Energy-Aware Inference
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Energy-Aware Inference.md
updated: 2026-07-24
---

# Energy-Aware Inference
[Energy-Aware Inference](https://www.notion.so/33ec72bad013818aa564f83af05fb2ba)
Concept
Inference must respect device power and thermal constraints.
Rule / Mechanism
Inference intensity adapts based on: - Battery level - Thermal state - Execution context (foreground vs background)
Non-essential inference is deferred under constraint.
Why It Exists
An intelligent system that drains power is not intelligent.
Implications
Reduced energy impact
Longer live sessions
Predictable performance
See also: [[On-Device First Principle]], [[Live Activity Inference]]

---

## Inference Speed — Prompt Efficiency Rules

System prompt size directly affects inference latency. Compact capability headers are preferred over verbose prose:

```json
{
  "capabilities": {
    "tier": "free|pro",
    "role": "user|trainer|admin",
    "domain": "planning|workout|nutrition|recovery",
    "agenticEnabled": false,
    "isActiveWorkout": false
  }
}
```

Target: ~70–80% reduction in system prompt tokens vs. prose-form policy overlays. Shorter prompts = fewer tokens to encode/decode = faster first-token latency on constrained devices.

### Mobile Model Efficiency (Phi-4-mini reference)

For Q4_K_M quantized models on constrained devices (≤6 GB RAM):
- Token budget per inference: constrained by available RAM after model loading
- Prompt length is the primary controllable cost factor
- Adapter stack size affects memory pressure — 1–2 adapters is the baseline; >4 adapters requires active RAM management

## Related

^[source-materials/mirrors/doctrine/Energy-Aware Inference.md]
