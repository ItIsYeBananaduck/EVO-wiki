---
title: Secure Prompted Entry
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Secure Prompted Entry.md
updated: 2026-07-24
---

# Secure Prompted Entry
[Secure Prompted Entry](https://www.notion.so/33ec72bad013812db510d658a91c70c9)
Concept
When Alice needs sensitive data, the user enters it via a secure UI outside the chat stream.
Rule / Mechanism
Alice requests a secret by type (e.g., password, SSN, API key)
App shows a secure popup/input sheet (not chat)
The secret is never inserted into conversation history
Delegator passes the secret directly to the tool executor at runtime
The model only receives a placeholder/ack (e.g., “SECRET_PROVIDED”)
Why It Exists
Fastest path to preventing secrets from leaking into chat logs.
Implications
Works without a full vault
Still requires careful logging discipline
Suitable for one-off secrets
Links
[Secret Isolation Rule](https://www.notion.so/33ec72bad013813e9eb6ead8af1141ad)
[No Secret Echo Rule](https://www.notion.so/33ec72bad013814892ccd1e5e72a397e)
Scoped Tool Grants

## Related

^[source-materials/mirrors/doctrine/Secure Prompted Entry.md]
