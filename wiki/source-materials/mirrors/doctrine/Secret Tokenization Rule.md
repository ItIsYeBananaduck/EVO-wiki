---
title: Secret Tokenization Rule
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Secret Tokenization Rule.md"]
updated: 2026-07-24
---

# Secret Tokenization Rule
[Secret Tokenization Rule](https://www.notion.so/33ec72bad013818fa0e9c5ea0c210546)
Concept
The model references secrets using tokens/handles, not plaintext.
Rule / Mechanism
The model may request “use credential X” or “use VAULT_TOKEN_Y”
The tool executor resolves the token to plaintext only at the moment of use
Token scope is bound to task + tool grant
Why It Exists
Tokenization prevents disclosure while still enabling automation.
Implications
Supports reuse safely
Limits blast radius via scoped grants
Enables auditing without leakage
Links
[Advanced Vault Protocol](https://www.notion.so/33ec72bad01381fb9609d935002a22a6)
Scoped Tool Grants

## Related
