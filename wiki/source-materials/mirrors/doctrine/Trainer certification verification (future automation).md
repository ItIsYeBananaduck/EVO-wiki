---
title: Trainer certification verification (future automation)
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Trainer certification verification (future automation).md"]
updated: 2026-07-24
---

# Trainer certification verification (future automation)
## Purpose

Capture the intended architecture stance for trainer certification verification.

## Current Status

Future work. Not required for MVP; manual process is acceptable until there is revenue or an explicit legal/compliance requirement.

## Current Understanding

- A Supabase Edge Function exists in the repo (`verify-trainer`) that attempts to validate trainer certifications (ACE/NASM via external APIs if configured; otherwise manual pending) and stores results in Supabase (`trainer_attributions`).
- It is currently **not wired** (no runtime call sites found).
- Even if/when implemented, this should be treated as an **onboarding/compliance workflow**, not a dependency for on-device workout execution.

## What Is Known

- Manual certification verification can meet early-stage needs.
- Automating verification is a “nice to have” and should not expand the critical runtime surface area.

## What Is Uncertain

- Whether certification validation is legally required for all trainers in target markets.
- Whether any external certification providers (ACE/NASM/etc.) have stable APIs suitable for production automation.

## What Needs Verification

- Actual legal/compliance requirements by market.
- Whether trainer verification is needed before any specific in-app trainer features can be enabled.

## Current Conclusion

Park automated certification validation as future work. Keep the product surface area small; if reintroduced, ensure it remains an external workflow boundary and does not become cloud-authoritative for core runtime state.

## Related

^[{src_rel}]
