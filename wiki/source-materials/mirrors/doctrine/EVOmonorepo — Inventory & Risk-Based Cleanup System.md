## Parent
- [[MOC - EVOmonorepo]]

## Related
- [[EVOconnect - Architecture]]
- [[EVOconnect - Delegator & Governance]]

---

# Purpose

> Maintain a clean, conflict-free codebase by separating:
> **inventory → risk → action → validation**

This system is designed to:
- identify leftover code from additive AI refactors
- detect conflicting implementations before they break the app
- reduce repo noise (tools, docs, legacy paths)
- enable safe, incremental cleanup

---

# Core Principles

- Inventory ≠ Cleanup  
- Stale ≠ Removable  
- Ownership must be singular  
- Cleanup must be reversible  
- Always validate after change  

---

# System Overview

## Inventory Map
Tracks what exists.

## Risk Map
Tracks what is dangerous or misleading.

## Cleanup Actions
Defines what should be done.

---

# Phase 1 — Inventory Mapping (Path Clusters)

Map the repo into **auditable clusters**, not individual files.

## For each cluster record:

- name  
- path  
- category  
- tech area  
- responsibility (what it appears to own)  
- initial status:
  - active
  - unclear
  - legacy

---

## Categories (discovered during mapping)

### Workflow Artifacts
- dev tools
- agent tooling residue
- planning/spec systems

### Docs
- audits
- plans
- specs
- generated docs

### Implementation Areas
- AI runtime
- federation / adapters
- Flutter app
- Svelte / web
- platform-native code
- shared packages
- infra/backend
- build/config

---

# Phase 2 — Risk Mapping

Evaluate each cluster for risk.

## Risk Statuses (code/systems)

- active  
- active but duplicated  
- shadow path  
- stale but harmless  
- conflict risk  
- migration scaffolding  
- decision required  
- purge candidate  

---

## Risk Statuses (docs)

- source of truth  
- active reference  
- historical  
- superseded  
- obsolete noise  
- archive candidate  

---

## Risk Statuses (dev tools)

- active workflow  
- fallback workflow  
- deprecated  
- residual  
- purge candidate  

---

## Key Questions

For each cluster:

- What does this own?  
- What is the active path?  
- Is there a duplicate or shadow path?  
- Could this conflict under edge cases?  
- Is this still referenced meaningfully?  
- Is this influencing current work incorrectly?  

---

# Phase 3 — Cleanup Action Layer

Every risky item must have an action:

- keep  
- consolidate  
- archive  
- decide  
- purge  
- refactor first  

---

## Rule

> A file is purgeable only if it is:
- unreferenced in meaningful execution paths
- non-critical
- non-transitional
- non-conflicting
- validated safe

---

# Phase 4 — Claude Validation

Before cleanup:

Provide:
- inventory map  
- risk map  
- cleanup plan  

Ask:

- what dependencies are missing?
- what could break?
- what conflicts were overlooked?

---

# Phase 5 — Execution (Surgical Cleanup)

Perform cleanup in **bounded slices**:

- clean one cluster or sub-cluster  
- run app / tests  
- validate behavior  
- continue  

---

# Parallel Execution Rules

## Safe to parallelize

- docs  
- dev tools  
- isolated packages  
- assets  

---

## Not safe to parallelize

- shared packages  
- runtime systems  
- adapter/federation logic  
- app entrypoints  
- cross-package dependencies  

---

# Critical Constraint

> Every responsibility must have ONE active owner

If multiple paths exist:
- mark as conflict risk or decision required

---

# Doc Handling Strategy

Docs must not mislead current work.

## Allowed states:

- active (source of truth)  
- reference  
- archived  
- deprecated  

---

## Rule

> Old docs may exist, but must never appear current

---

# Dev Tool Strategy

Dev tools are tied to workflow evolution.

## Allowed states:

- active  
- deprecated  
- residual  
- purge candidate  

---

## Rule

> If a tool is not part of the current workflow, it must not influence the repo

---

# Output of System

Each audit produces:

- categorized inventory  
- risk classification  
- cleanup actions  

These combine into a **cleanup plan**

---

# Long-Term Use

This system is reusable for:

- post-refactor cleanup  
- pre-release stabilization  
- architecture migrations  
- agent-driven development drift correction  

---

# Core Takeaway

> Map first  
> Identify risk second  
> Decide third  
> Clean safely  
> Validate always