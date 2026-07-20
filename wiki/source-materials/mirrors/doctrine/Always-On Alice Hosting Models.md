## Core Question

Can users run an always-on or near-always-available Alice without EVO hosting it?

**Answer: Yes — but only with clearly separated hosting models and strict boundaries.**

---

## Why This Matters

Connect is:
- a control layer
- a hive orchestrator
- a bounded automation environment

It must NOT become:
- an unrestricted agent
- a cloud-controlled system

---

## Model 1 — Standard Local Runtime

### Description
Alice runs inside the EVO app.

### Pros
- safest
- simplest
- fully governed

### Cons
- limited background execution (especially mobile)
- not suitable for always-on

---

## Model 2 — Self-Hosted Alice Service

### Description
User runs Alice on their own machine.

### Examples
- Docker container
- local service
- Mac mini / PC / NAS

### Purpose
- persistent node
- Hive anchor
- orchestration layer

### Key Rule
Docker is **packaging**, not security.

### Responsibilities
- task coordination
- plugin access
- Hive participation

---

## Model 3 — Privileged Helper (Supervised Execution)

### Description
Separate minimal system component for elevated actions.

### Key Idea
Alice does NOT run as admin.

Instead:
- Alice = planner
- Helper = executor

### Rules
- narrow scope
- non-agentic
- fully logged
- explicit approval required

---

## Optional Model — Alice as OS User

### Description
Alice runs under a separate OS account.

### Benefits
- isolation
- clean workspace
- safer automation boundary

### Risks
- still needs governance
- not sufficient alone

---

## Recommended Architecture

### Layer 1 — Alice Runtime
- planning
- orchestration
- task handling

### Layer 2 — Capability Broker
- permission checks
- routing decisions
- Delegator enforcement

### Layer 3 — Execution Surfaces
- app runtime
- Docker service
- terminal sandbox
- privileged helper

### Layer 4 — Logging
- task id
- method id
- action
- result

---

## V1 Recommendation

Start with:

- Desktop anchor runtime
- Standard user privileges
- Optional Docker packaging
- No admin access
- Bounded execution only

Then add:

- narrow privileged helper

Later:

- optional Alice user profile

---

## Core Takeaway

> Alice should never run as unrestricted admin.

Instead:

> Standard runtime + bounded helpers + explicit permissions = safe power