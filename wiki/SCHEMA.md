---
title: "EVO Wiki Schema"
type: page
tags: ['lsctech', 'evo']
updated: 2026-07-20
---

# EVO Wiki Schema

## Domain
EVO ecosystem doctrine and Polaris-managed canonical docs.

## Conventions
- File names: lowercase, hyphens, no spaces unless matching original doctrine title.
- Curated pages live in `doctrine/`, `canonical/`, and `components/`.
- Source mirrors live under `source-materials/mirrors/`. These are symlinks or copies and should not be edited here.
- Use relative `[[wikilinks]]` between curated pages.
- Every curated page must list relevant mirror sources.
- Tags come from this taxonomy:
  - Topics: doctrine, architecture, governance, escalation, accountability, intelligence
  - Roles: teacher, student, alice, guardian, trainer, learner
  - Surfaces: connect, learn, terminal, browser, hive, vault, mobile
  - Meta: canonical, audit, decision, spec, deprecated
- Do not duplicate doctrine. Curated pages synthesize mirror content.
- Every new curated page must be linked from `index.md`.

## Sources
- EVO doctrine originals: `/Users/lsctech/EVO/` mirrored into `source-materials/mirrors/doctrine/`.
- EVO canonical docs: `smartdocs/canonical/` mirrored into `source-materials/mirrors/canonical/`.
- Do not ingest `smartdocs/doctrine/active/` bulk; it is derivative of EVO doctrine.