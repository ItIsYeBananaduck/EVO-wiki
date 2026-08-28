---
title: "Evoconnect — Delegator Model"
type: doctrine
tags: ['lsctech', 'doctrine', 'source-material', 'evo']
updated: 2026-08-24
---

## Concept

Delegator is the **authority layer** of EVOconnect.

It enforces:
- execution boundaries  
- approvals  
- policy compliance  
- system safety  

Delegator determines **what Alice is allowed to do**.

---

## Responsibilities

- validate all actions before execution  
- enforce Method and Talent constraints  
- control access to Browser and Terminal  
- enforce Vault policies  
- detect and stop deviations  
- terminate unsafe execution immediately  

---

## Authority Model

Delegator overrides:

- Alice reasoning  
- webpage instructions  
- terminal output  
- inferred intent  

---

### 🔑 Law

> Delegator is the final authority over all execution.

---

## Behavior

Delegator ensures:

- no action occurs without approval  
- no scope expansion occurs  
- no deviation from Method/Talent  
- no unauthorized secret usage  

---

## Related Concepts

- [[EVOconnect — Execution Model]]
- [[EVOconnect — Browser & Terminal Execution Model]]
- [[EVOconnect — Business Safety Guarantees]]
- [[EVOconnect — Vault Model]]