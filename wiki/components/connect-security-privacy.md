---
title: Connect — Security & Privacy Model
type: concept
tags: [evo, connect, security, privacy, secret, vault]
sources:
  - source-materials/mirrors/doctrine/Connect - Security & Privacy Model.md
  - source-materials/mirrors/doctrine/Advanced Vault Protocol.md
updated: 2026-07-20
---

# Connect — Security & Privacy Model

## Trust Boundary
Connect owns platform trust boundaries. Training data, journals, and LoRAs remain in app-local or user-cloud storage.

## Secret Model
Secrets are tokenized. Secret access paths require authorization and cannot be influenced by untrusted content.

## Vault Usage
Advanced vault protocol defines minimal exposure rules: secrets are resolved only through explicit grant paths.

## Related
- [[Connect — Task Control Plane]]
- [[Alice Capability Boundary]]

^[source-materials/mirrors/doctrine/Connect - Security & Privacy Model.md]
^[source-materials/mirrors/doctrine/Advanced Vault Protocol.md]
