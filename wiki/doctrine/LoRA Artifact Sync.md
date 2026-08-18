---
title: LoRA Artifact Sync
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/LoRA Artifact Sync.md"]
updated: 2026-07-24
---

# LoRA Artifact Sync
[LoRA Artifact Sync](https://www.notion.so/33ec72bad0138155b304efaefc565899)
Concept
All devices in the Hive must share the same LoRA set and versions to preserve consistent behavior.
Rule / Mechanism
Hive synchronizes LoRA artifacts as versioned packages: - LoRA id + semantic version - compatibility metadata (base model / quant / format) - signature / integrity check
Devices may run different hardware, but must converge on the same effective LoRA set.
Why It Exists
If devices diverge in LoRAs, Alice becomes inconsistent across devices.
Implications
LoRAs are treated as first-class synced artifacts
Sync must be resumable and verifiable
Updates must be atomic from the user’s perspective
Links
[Hive Shared State Backbone](https://www.notion.so/33ec72bad01381a2845acb86cd36ef48)

---

Related notes: [[EVOLoRA Mesh — Adapter Creation Pipeline]]

## Related

^[{src_rel}]
