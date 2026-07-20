# Secret Tokenization Rule

## Concept
The model references secrets using tokens/handles, not plaintext.

## Rule / Mechanism
- The model may request "use credential X" or "use VAULT_TOKEN_Y"
- The tool executor resolves the token to plaintext only at the moment of use
- Token scope is bound to task + tool grant

## Why It Exists
Tokenization prevents disclosure while still enabling automation.

## Implications
- Supports reuse safely
- Limits blast radius via scoped grants
- Enables auditing without leakage

## Links
- [[Advanced Vault Protocol]]
- [[Scoped Tool Grants]]