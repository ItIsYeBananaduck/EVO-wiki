
## Purpose
Define how Alice interacts with external tools and environments when direct provider integration is unavailable.

This layer enables Alice to:
- Use local desktop tools
- Interact with human-facing AI apps (Claude, etc.)
- Execute governed automation
- Maintain full auditability and safety

---

## Core Idea

Not all intelligence is accessible via API or OAuth.

Instead of forcing provider integration, Connect supports:
- Provider integrations (OAuth/API)
- Environment integrations (EVOterminal)
- Manual fallback workflows

---

## Principles

- Local-first execution  
- No fake integrations  
- Environment access is governed  
- No unrestricted automation  
- All actions must be auditable  
- User remains in control  

---

## Integration Types

### 1. Provider Integrations
- OAuth providers
- API providers
- Local models

### 2. Environment Integrations
- EVOterminal
- Internal browser
- Desktop applications

### 3. Fallback Integrations
- Manual escalation packet export
- Copy/paste workflows

---

## Role of EVOterminal

EVOterminal is:
- A governed execution surface  
- A bridge to local tools and apps  
- A controlled interface for Computer Alice  
- Not a raw terminal or unrestricted shell  

---

## Connected Systems

- [[Connect - Hive Architecture]]
- [[Connect - Delegator & Governance]]
- [[Connect - Task System]]
- [[Connect - Control Panel & Tools]]

---

## Child Notes

- [[EVOterminal - Core Design]]
- [[Environment Integrations]]
- [[Desktop Orchestration via Hive]]
- [[Provider vs Environment Access]]
#connect