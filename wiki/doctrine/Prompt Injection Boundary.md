---
title: Prompt Injection Boundary
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Prompt Injection Boundary.md"]
updated: 2026-07-24
---

# Prompt Injection Boundary
[Prompt Injection Boundary](https://www.notion.so/33ec72bad01381ce8e47d760c92c69a1)
Concept
Untrusted content (especially web pages) must not be allowed to influence secret access.
Rule / Mechanism
Web content is treated as untrusted input. Secret access requires: - a valid task authorization path - a valid tool grant - explicit token usage and cannot be triggered by instructions found in web content.
Why It Exists
Prompt injection is a primary exfiltration risk in browser automation.
Implications
Safety browser wrapper must isolate instructions from content
Secret tokens cannot be resolved due to page content requests
Links
[Safety Browser Protocol](https://www.notion.so/33ec72bad0138181a57adc668e4b726c)
[Secret Tokenization Rule](https://www.notion.so/33ec72bad013818fa0e9c5ea0c210546)
[Delegator Tool Hostage Rule](https://www.notion.so/33ec72bad013817d9b6bcc64f4a096fd)
[Method Non-Deviation Rule](https://www.notion.so/33ec72bad01381a7b5c9c709be5646ba)
[Whitelisted Instruction Sources](https://www.notion.so/33ec72bad01381db94dbc683f4e90150)

## Related

^[{src_rel}]
