# Advanced Vault Protocol

## Concept
A vault stores any user-designated sensitive information (not just passwords) and provides it by token at execution time.

## Rule / Mechanism
- Vault stores encrypted secrets at rest
- Model never receives plaintext secrets
- Model references secrets by token/handle (e.g., VAULT_TOKEN_123)
- Only tool executor resolves token → plaintext, in-memory, just-in-time
- Secret values are never logged; only token references are logged

## Why It Exists
Enables safe reuse, automation, and consistency across tasks without ever exposing secrets to the model.

## Implications
- Enables “learned” repeatable tasks with secrets
- Centralizes sensitive data governance
- Supports per-secret permissions and revocation

## Links
- [[Secret Tokenization Rule]]
- [[No Secret Echo Rule]]
- [[Vault Retention & Deletion]]