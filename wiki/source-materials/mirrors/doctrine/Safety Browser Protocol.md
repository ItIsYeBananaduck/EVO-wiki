---
title: Safety Browser Protocol
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Safety Browser Protocol.md"]
updated: 2026-07-24
---

# Safety Browser Protocol
[Safety Browser Protocol](https://www.notion.so/33ec72bad0138181a57adc668e4b726c)
Concept
Sensitive workflows should be executed inside a controlled browser sandbox with a privacy wrapper.
Rule / Mechanism
Alice performs actions in a controlled browser environment
The wrapper detects sensitive fields and prevents secrets from appearing in model-visible text
User can intervene at any time
The browser layer can request secrets via the vault using tokens/handles
Why It Exists
Many real tasks require web workflows where secrets may appear on-screen or in form fields.
Implications
Prevents prompt injection via web content from exfiltrating secrets
Makes “AI does things on my behalf” feasible and auditable
Links
[Advanced Vault Protocol](https://www.notion.so/33ec72bad01381fb9609d935002a22a6)
[Secret Isolation Rule](https://www.notion.so/33ec72bad013813e9eb6ead8af1141ad)
[Prompt Injection Boundary](https://www.notion.so/33ec72bad01381ce8e47d760c92c69a1)

## Related
