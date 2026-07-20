# EVE – LoRA Strategy (Generic vs Starter)

EVE domains are implemented as LoRAs.

Two deployment models:

---

## 1) Generic EVE (Core-only)
- ships quickly
- low cost
- domain-agnostic
- provides basic ingestion + reporting

EVE improves by creating org-specific LoRAs over time based on local data.

Pros:
- cheapest
- fastest to deploy
- aligns with local-first vision

Cons:
- slower time-to-value
- "dumb at launch"

---

## 2) EVE + Starter LoRA (Seeded)
LSCT builds an initial LoRA using real org data and domain ontology.

Pros:
- fast time-to-value
- more relevant insights immediately
- smoother adoption

Cons:
- higher cost
- requires secure onboarding + dataset prep

---

## Governance: LoRA Training Work Orders
EVE can propose LoRA training jobs, but humans approve:
- data scope + retention
- objective metrics
- evaluation set
- rollback plan
- privacy constraints

No autonomous unreviewed LoRA creation.