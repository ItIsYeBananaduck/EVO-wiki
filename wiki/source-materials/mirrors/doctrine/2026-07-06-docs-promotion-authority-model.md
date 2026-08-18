---
title: 2026-07-06-docs-promotion-authority-model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/2026-07-06-docs-promotion-authority-model.md"]
updated: 2026-07-24
---

# Docs Promotion Authority Model

## Rule

Promotion of a SmartDocs candidate to `doctrine/active/` or `specs/active/` requires
explicit user confirmation in the session before the promoting call
(`polaris doctrine spec-promote <path> --approve`) may run. A conflict report must be
surfaced first (`polaris doctrine spec-promote <path>` without `--approve`).

Promotion to `architecture/` or `decisions/` is never performed by `docs-promote` at
all — those require the explicit ADR process, regardless of user confirmation.

## Evidence

This is not an inference or an imported convention. It is the actual, current behavior
of this repository's `@lsctech/polaris` CLI installation, confirmed by running
`npx polaris skill packet promote` in this repo on 2026-07-06. The returned packet's
`prohibited_actions` include, verbatim:

- "Call --approve without explicit user confirmation in the session"
- "Promote to architecture/ or decisions/ — those require explicit ADR process"

(A separate repo, `polaris-clo-smartdocs` — Polaris's own product cognition bundle —
documents a similar-sounding rule in its own `docs-authority-model.md`. That is a
different repository's spec and was previously the only source cited for this rule in
git-fit's own drift-remediation spec. This doctrine file replaces that citation with
git-fit-local, CLI-verified evidence.)

## Why this matters

Before this file existed, an agent working in git-fit had no local doctrine to consult
for "does promoting a doc require approval here?" — the answer was only discoverable
by invoking the CLI and reading its response, or by incorrectly assuming a different
repo's spec applies here. This file makes the answer discoverable via normal doctrine
navigation (`smartdocs/doctrine/active/`) without requiring a CLI round-trip.

## Related

- `.polaris/skills/docs-promote/chain.md` — the operational chain that enforces this
  rule step-by-step (step `05-await-approval`).
- `.polaris/skills/docs-promote/SKILL.md` — the skill definition mirroring the same
  `prohibited_actions`.
^[{src_rel}]
