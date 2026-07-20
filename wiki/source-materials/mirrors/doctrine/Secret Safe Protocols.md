# Secret Safe Protocols

## Concept
Sensitive user data must never flow through the chat channel or model-visible context.

## Rule / Mechanism
Two protocols exist:
1) Secure Prompted Entry (temporary)
2) Safety Browser + Advanced Vault (long-term)

Both protocols ensure the model only receives non-sensitive tokens/handles.

## Why It Exists
Secrets in chat become unrecoverable liabilities (logging, memory, prompt injection, accidental exposure).

## Implications
- Secrets are handled by tools, not language output
- Delegator must enforce secret isolation
- Audit logs must avoid plaintext secrets

## Links
- [[Secure Prompted Entry]]
- [[Safety Browser Protocol]]
- [[Advanced Vault Protocol]]
- [[Delegator Tool Hostage Rule]]