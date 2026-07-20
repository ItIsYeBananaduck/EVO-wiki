## Parent
- [[MOC - EVOconnect (Modular OS Layer)]]
## Purpose
Define how Alice:
- remains present across all surfaces
- respects strict learning boundaries
- safely executes privileged (sudo-level) actions
- integrates with Delegator, Vault, and Task System

This MOC unifies:
- Awareness vs Learning
- Training Mode
- Privileged Execution
- Vault-based credential access
- Recurring governed automation

---

## Core Concepts

- [[Awareness vs Learning Boundary]]
- [[EVOconnect — Talent Training Mode (Intentional Learning) 1]]
- [[Privileged Execution Model]]
- [[Vault-Based Credential Access]]
- [[Recurring Talents & Scheduled Execution]]

---

## System Relationships

### Governance Layer
- [[Connect - Delegator & Governance]]

### Execution Layer
- [[EVOterminal - Core Design]]
- [[Provider vs Environment Access]]

### Task System
- [[Connect - Task System]]

### Security Layer
- [[Connect - Security & Privacy Model]]

---

## Key Principles

1. Alice is always present, but never invasive  
2. Learning requires explicit user intent  
3. Privileged actions require explicit approval  
4. Credentials are never exposed, only used via vault  
5. All execution is governed, logged, and auditable  

---

## Design Boundaries

### Awareness
- Always active  
- No learning  
- No memory creation  

### Learning
- Training Mode only  
- Method creation required  
- User approval required  

### Execution
- Must bind to Method  
- Must pass Delegator  
- Must be logged  

### Privileged Execution
- Requires explicit approval  
- Vault-mediated  
- No silent escalation  

---

## Example Flow

### System Cleanup Talent

1. User enters Training Mode  
2. Performs cleanup via EVOterminal  
3. Alice constructs Method  
4. User approves Talent  
5. User schedules weekly execution  
6. Future runs:
   - prompt for sudo approval (unless whitelisted)
   - execute via EVOterminal
   - log results  

---

## Future Extensions

- Trust tiers for privileged Talents  
- Adaptive approval suggestions  
- Storage-aware system monitoring  
- Auto-suggested cleanup (non-learning awareness)  

---

## Core Takeaway

> Awareness gives Alice presence.  
> Training Mode gives Alice growth.  
> Delegator ensures Alice remains safe.  
> Vault ensures Alice never becomes dangerous.

#connect #cognition