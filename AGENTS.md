---
title: "EVO-wiki Agent Rules"
type: page
tags: ['lsctech', 'evo']
updated: 2026-08-24
---

# EVO-wiki Agent Rules

## Wiki Doctrine Ingestion
This repo contains wiki doctrine. Any promotion, normalization, or curation of doctrine
content MUST follow `wiki/WIKI_INGESTION_GUIDE.md`.

Key rules:
- Both `wiki/doctrine/` and `wiki/source-materials/mirrors/doctrine/` must stay identical to `lsctech-wiki`.
- Every doctrine file needs OKF frontmatter, `## Related`, and `^[...]` provenance.
- Use repo-relative paths only. No `/Users/lsctech/...` absolute paths.
- `index.md` is infrastructure, not doctrine. Append rows when promoting; do not rewrite existing rows.
- `_lifecycle-manifest.md` is infrastructure. Prefer removing it from `wiki/doctrine/`.
- Do not promote implementation specs, audit reports, crash fixes, or `SUPERTONIC_2_*` files.

When in doubt, leave it out and flag for human review.
