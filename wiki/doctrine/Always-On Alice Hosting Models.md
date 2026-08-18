---
title: Always-On Alice Hosting Models
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Always-On Alice Hosting Models.md"]
updated: 2026-07-24
---

# Always-On Alice Hosting Models
Core Question
Can users run an always-on or near-always-available Alice without EVO hosting it?
Answer: Yes — but only with clearly separated hosting models and strict boundaries.

Why This Matters
Connect is: - a control layer - a hive orchestrator - a bounded automation environment
It must NOT become: - an unrestricted agent - a cloud-controlled system

Model 1 — Standard Local Runtime
Description
Alice runs inside the EVO app.
Pros
safest
simplest
fully governed
Cons
limited background execution (especially mobile)
not suitable for always-on

Model 2 — Self-Hosted Alice Service
Description
User runs Alice on their own machine.
Examples
Docker container
local service
Mac mini / PC / NAS
Purpose
persistent node
Hive anchor
orchestration layer
Key Rule
Docker is packaging, not security.
Responsibilities
task coordination
plugin access
Hive participation

Model 3 — Privileged Helper (Supervised Execution)
Description
Separate minimal system component for elevated actions.
Key Idea
Alice does NOT run as admin.
Instead: - Alice = planner - Helper = executor
Rules
narrow scope
non-agentic
fully logged
explicit approval required

Optional Model — Alice as OS User
Description
Alice runs under a separate OS account.
Benefits
isolation
clean workspace
safer automation boundary
Risks
still needs governance
not sufficient alone

Recommended Architecture
Layer 1 — Alice Runtime
planning
orchestration
task handling
Layer 2 — Capability Broker
permission checks
routing decisions
Delegator enforcement
Layer 3 — Execution Surfaces
app runtime
Docker service
terminal sandbox
privileged helper
Layer 4 — Logging
task id
method id
action
result

V1 Recommendation
Start with:
Desktop anchor runtime
Standard user privileges
Optional Docker packaging
No admin access
Bounded execution only
Then add:
narrow privileged helper
Later:
optional Alice user profile

Core Takeaway
Alice should never run as unrestricted admin.
Instead:
Standard runtime + bounded helpers + explicit permissions = safe power

---

## Shared Runtime Ownership — Resident Alice Lifecycle

When domains (`mind`, `learn`, `connect`, `training`) coexist in a single app, one domain owns the resident Alice runtime at any given time.

### Lifecycle Rules (EVOS1-42)

- **Single runtime owner** at all time — no concurrent domain ownership of the Alice inference instance
- On domain switch, the outgoing domain must release its inference context before the incoming domain activates Alice
- A **handoff snapshot** is written to shared container (resume token + continuation state) before release
- The incoming domain may resume the snapshot or start a fresh session — but must declare intent explicitly
- If the outgoing domain cannot release within the handoff window, the new domain starts a clean session and the prior context is expired

### Fallback Unload

If no domain claims ownership within `unloadTimeoutMs`, Alice transitions to standby:
- inference session suspended
- context snapshot preserved with TTL
- Alice becomes re-activatable but does not hold runtime resources

### App/Process Boundary Extension

When domains become separate app bundles in the future, the same lifecycle rules apply via the durable IPC queue described in [[IPC Strategy — Multi-App Domain Separation]]. The handoff snapshot becomes the cross-process transfer unit.

## Related

^[{src_rel}]
