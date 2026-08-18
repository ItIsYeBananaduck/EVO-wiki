---
title: Secret Isolation Rule
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Secret Isolation Rule.md"]
updated: 2026-07-24
---

# Secret Isolation Rule
[Secret Isolation Rule](https://www.notion.so/33ec72bad013813e9eb6ead8af1141ad)
Concept
Sensitive values must be isolated from model-visible channels.
Rule / Mechanism
Secrets must never appear in: - chat transcript - prompt context - model outputs - tool call logs
Secrets may exist only in: - secure UI inputs - encrypted vault storage - short-lived execution memory
Why It Exists
Model-visible secrets are effectively leaked.
Implications
Strong boundary between “chat” and “execution”
Requires strict logging and telemetry filtering
Links
[Secret Safe Protocols](https://www.notion.so/33ec72bad01381bdb96afc2bdc69d955)
[No Secret Echo Rule](https://www.notion.so/33ec72bad013814892ccd1e5e72a397e)

## Related

^[{src_rel}]
