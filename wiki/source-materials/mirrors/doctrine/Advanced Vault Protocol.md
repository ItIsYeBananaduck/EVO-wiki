---
title: Advanced Vault Protocol
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Advanced Vault Protocol.md"]
updated: 2026-07-24
---

# Advanced Vault Protocol
[Advanced Vault Protocol](https://www.notion.so/33ec72bad01381fb9609d935002a22a6)
Concept
A vault stores any user-designated sensitive information (not just passwords) and provides it by token at execution time.
Rule / Mechanism
Vault stores encrypted secrets at rest
Model never receives plaintext secrets
Model references secrets by token/handle (e.g., VAULT_TOKEN_123)
Only tool executor resolves token → plaintext, in-memory, just-in-time
Secret values are never logged; only token references are logged
Why It Exists
Enables safe reuse, automation, and consistency across tasks without ever exposing secrets to the model.
Implications
Enables “learned” repeatable tasks with secrets
Centralizes sensitive data governance
Supports per-secret permissions and revocation
Links
[Secret Tokenization Rule](https://www.notion.so/33ec72bad013818fa0e9c5ea0c210546)
[No Secret Echo Rule](https://www.notion.so/33ec72bad013814892ccd1e5e72a397e)
[Advanced Vault Protocol](https://www.notion.so/33ec72bad01381fb9609d935002a22a6)

## Related
