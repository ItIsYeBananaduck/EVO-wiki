---
title: Provider vs Environment Access
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Provider vs Environment Access.md"]
updated: 2026-07-24
---

# Provider vs Environment Access
Purpose
Define the distinction between direct integrations and environment-based access.

Provider Access
Definition
Direct connection to a service via: - OAuth- API- local model
Characteristics
Structured inputs/outputs
Reliable automation
Governed execution
Scalable

Environment Access
Definition
Indirect interaction through: - EVOterminal- browser- desktop apps
Characteristics
Human-oriented interfaces
Less structured
Requires orchestration
May require user involvement

When to Use Each
Use Provider Access When:
API or OAuth exists
Structured responses are needed
automation is reliable
Use Environment Access When:
no API exists
OAuth is restricted
tool is human-facing
user already has access

Fallback Hierarchy
Local model
OAuth provider
API provider
Environment integration
Manual fallback

Key Principle
Do not force environment tools into provider models.
Respect: - provider boundaries- user ownership- system governance

[Philosophy](https://www.notion.so/33ec72bad0138193a0bbce9f89d79395)
Alice does not need direct control of everything.
She needs: - the ability to orchestrate everything

## Related

^[{src_rel}]
