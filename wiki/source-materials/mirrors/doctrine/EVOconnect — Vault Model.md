---
tags:
  - concept/vault
  - concept/secrets
  - concept/security
  - concept/data
  - concept/governance
  - type/concept
  - type/architecture
---

## Concept

The Vault is a **policy-controlled secret system**, not just storage.

It governs:
- how secrets are stored  
- where secrets can be used  
- how secrets are injected  
- when user approval is required  

---

## Secret Types

- credentials (passwords, API keys)  
- identity fields (address, SSN, DOB)  
- financial data (bank, payout info)  
- site-bound reusable fields  

---

## Secret Model

Each secret is defined by:

- origin (allowed domains)  
- field type (where it applies)  
- action (inject / reveal / user-entry)  
- approval level (auto / confirm / required)  

---

### 🔑 Law

> Secrets are not data. They are permissions.

---

## Secret Actions

- inject (default)  
- reveal (restricted)  
- suggest (requires approval)  
- user-only entry (highest safety)  

---

## Behavior

- secrets are never freely exposed  
- secrets cannot be copied or summarized  
- secrets cannot be requested by webpages  
- all usage is logged  

---

## Related Concepts

- [[EVOconnect — Business Safety Guarantees]]
- [[EVOconnect — Browser & Terminal Execution Model]]
- [[EVOconnect — Delegator Model]]