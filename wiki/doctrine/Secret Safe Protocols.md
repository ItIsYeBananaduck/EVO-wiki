---
title: Secret Safe Protocols
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Secret Safe Protocols.md
updated: 2026-07-24
---

# Secret Safe Protocols
[Secret Safe Protocols](https://www.notion.so/33ec72bad01381bdb96afc2bdc69d955)
Concept
Sensitive user data must never flow through the chat channel or model-visible context.
Rule / Mechanism
Two protocols exist: 1) Secure Prompted Entry (temporary) 2) Safety Browser + Advanced Vault (long-term)
Both protocols ensure the model only receives non-sensitive tokens/handles.
Why It Exists
Secrets in chat become unrecoverable liabilities (logging, memory, prompt injection, accidental exposure).
Implications
Secrets are handled by tools, not language output
Delegator must enforce secret isolation
Audit logs must avoid plaintext secrets
Links
[Secure Prompted Entry](https://www.notion.so/33ec72bad013812db510d658a91c70c9)
[Safety Browser Protocol](https://www.notion.so/33ec72bad0138181a57adc668e4b726c)
[Advanced Vault Protocol](https://www.notion.so/33ec72bad01381fb9609d935002a22a6)
[Delegator Tool Hostage Rule](https://www.notion.so/33ec72bad013817d9b6bcc64f4a096fd)

## Related

^[source-materials/mirrors/doctrine/Secret Safe Protocols.md]
