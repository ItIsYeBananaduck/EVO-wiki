---
title: Echo_3_Security_Activation
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Echo_3_Security_Activation.md"]
updated: 2026-07-24
---

# Echo_3_Security_Activation
EVOmind — Echo: Security & Activation Model
Security Model:
- Per-recipient encryption
- No cross-recipient visibility
- Local-first decryption
- Version-controlled bundles

Threats:
- Premature unlock
- Data leakage
- Bundle theft

Activation Model:
Stage 1: Inactivity → eligibility
Stage 2: Recipient request (key)
Stage 3: Source challenge (email/app)
Stage 4: Grace period
Stage 5: Unlock if no response

Rule:
Inactivity does not unlock Echo.

Challenge Requirement:
User must click link and sign in to cancel unlock.

Unlock Rule:
Eligibility + key + unanswered challenge = unlock

Multi-Echo:
Each Echo has independent unlock state.

## Related

^[{src_rel}]
