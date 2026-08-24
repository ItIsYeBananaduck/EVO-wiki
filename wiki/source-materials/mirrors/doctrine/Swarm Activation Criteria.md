---
title: Swarm Activation Criteria
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Swarm Activation Criteria.md"]
updated: 2026-07-24
---

# Swarm Activation Criteria
[Swarm Activation Criteria](https://www.notion.so/33ec72bad013811faa5bd25064a5df78)
Concept
Swarm mode is triggered only when local single-node compute is insufficient or undesirable.
Rule / Mechanism
Swarm may be activated if: - estimated inference cost exceeds the lease holder budget, AND - no single device can complete within constraints, OR - parallel execution is significantly safer (thermal/battery) than single-node.
Swarm must not activate for trivial prompts.
Why It Exists
Swarm is powerful but expensive; it should be reserved for heavy work.
Implications
Predictable energy use
Reduced unnecessary complexity
Links
Inference Budget Ceiling
[Energy-Aware Inference](https://www.notion.so/33ec72bad013818aa564f83af05fb2ba)
[Swarm Parallel Inference](https://www.notion.so/33ec72bad013810eaa8ac7df235f10d0)

## Related
