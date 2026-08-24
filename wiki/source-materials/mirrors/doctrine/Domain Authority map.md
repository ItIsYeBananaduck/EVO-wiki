---
title: Domain Authority map
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Domain Authority map.md"]
updated: 2026-07-24
---

# Domain Authority map
## Domain Scope Map

| Domain | Authority scope |
|---|---|
| **training** | Physical performance, recovery, load progression |
| **mind** | Stress awareness, journaling, regulation prompts, emotional pattern observation |
| **learn** | Knowledge acquisition, cognitive skill development, retention and mastery tracking |
| **connect** (hub) | Task execution, agentic workflows, delegation |

**Rule**: No domain may assume authority outside its defined scope. Cross-domain influence may affect tone and timing only.

---

## Domain Package Contract (target architecture)

> This reflects the target directory structure, not the current repo layout. `mind`, `learn`, and `connect` packages do not yet exist; `training` diverges from this layout. Treat as the intended destination.

Each domain lives in `packages/domains/<domain>/` and provides:

```text
lib/src/
  capability_map.dart     # domain capability definitions
  guardrails.dart         # guardrail profile + rule IDs
  executors/              # domain executor implementations
  lora/
    lora_profile.dart     # LoRA kinds, weights, fallback policy
```

Shared runtime concerns (`ai-runtime`, `delegator`, `tools`, `mesh`) remain outside domain packages. Domain packages supply overlays/policies only — no direct inference hosting.

### LoRA Mapping per Domain

Each domain package declares which [[EVOLoRA Mesh]] adapter kinds are active for that domain and what fallback policy applies when adapters are missing or below confidence threshold.

## Related
