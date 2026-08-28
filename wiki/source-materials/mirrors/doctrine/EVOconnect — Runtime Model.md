---
title: "Evoconnect — Runtime Model"
type: doctrine
tags: ['lsctech', 'doctrine', 'source-material', 'evo']
updated: 2026-08-24
---

## Concept

The Runtime Model defines **where and how Alice operates across devices**.

---

## Device Roles

### Main Device
- user interaction  
- UI and communication  
- lightweight execution  

---

### Anchor Device
- primary compute  
- runs Connect continuously  
- wakes Alice when needed  

---

### Hive Devices (future)
- assist with compute  
- offload heavy tasks  
- distributed execution  

---

## Runtime States

### Idle
- minimal resource usage  
- Connect active  
- Alice inactive  

---

### Active
- Alice running  
- task execution  

---

### Cooling
- resources reduced  
- model unloaded or scaled down  

---

## Behavior Rules

- Alice is not always running  
- Connect is always available  
- compute adapts to device capacity  
- user experience is never degraded  

---

### 🔑 Law

> Alice must never degrade the user’s device experience.

---

## Scaling Model

- stronger device → stronger Alice  
- multi-device → distributed compute  
- fallback → local execution  

---

## Related Concepts

- [[EVOconnect — Execution Model]]
- [[EVOconnect — Delegator Model]]
- [[EVOconnect — Browser & Terminal Execution Model]]