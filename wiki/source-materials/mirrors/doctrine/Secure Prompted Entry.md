# Secure Prompted Entry

## Concept
When Alice needs sensitive data, the user enters it via a secure UI outside the chat stream.

## Rule / Mechanism
- Alice requests a secret by type (e.g., password, SSN, API key)
- App shows a secure popup/input sheet (not chat)
- The secret is never inserted into conversation history
- Delegator passes the secret directly to the tool executor at runtime
- The model only receives a placeholder/ack (e.g., "SECRET_PROVIDED")

## Why It Exists
Fastest path to preventing secrets from leaking into chat logs.

## Implications
- Works without a full vault
- Still requires careful logging discipline
- Suitable for one-off secrets

## Links
- [[Secret Isolation Rule]]
- [[No Secret Echo Rule]]
- [[Scoped Tool Grants]]