# Wiki Doctrine Ingestion Guide

## Purpose
This file is the authoritative instruction set for promoting EVO doctrine into the wiki. Follow these rules exactly when ingesting, normalizing, or auditing doctrine content.

---

## Source of Truth
All doctrine sources live in the EVO repo under:
- `EVO/smartdocs/raw/deprecated/doctrine-mirrors/`
- `EVO/smartdocs/raw/deprecated/doctrine-unmatched/`
- `EVO/smartdocs/raw/deprecated/doctrine-deprecated/`

Do NOT ingest from `EVO/smartdocs/canonical/audits/findings/` or `EVO/smartdocs/canonical/specs/` — those are implementation artifacts, not doctrine.

---

## Target Folders
Every promoted doctrine file must exist in **both** wikis, in these two locations:
- `lsctech-wiki/EVO/wiki/doctrine/`
- `EVO-wiki/wiki/doctrine/`

And their raw sources must be mirrored here:
- `lsctech-wiki/EVO/wiki/source-materials/mirrors/doctrine/`
- `EVO-wiki/wiki/source-materials/mirrors/doctrine/`

Both wikis must remain identical. Any write to one wiki must be duplicated to the other in the same change.

---

## OKF Format Requirements
Every doctrine file MUST contain:

1. **Frontmatter block** at the very top:
   ```
   ---
   title: <Clean Title Without "Raw Draft" or archive markers>
   type: concept
   tags: ["EVO","doctrine"]
   sources: ["<repo-relative path to source>"]
   updated: <YYYY-MM-DD>
   ---
   ```

2. **`## Related` section** with `[[wikilinks]]` to related doctrine files.

3. **Provenance footer** on its own line at the end:
   ```
   ^[<repo-relative source path>]
   ```

4. **No absolute local paths** anywhere in the file body. Use repo-relative paths only.

5. **No archive/promotion markers** like "promoted on 2026-05-12" or "archived".

---

## Promotion Rules
- Only promote files that are **doctrine** — principles, models, boundaries, rules, governance, escalation, talent, method, policy, protocol, guardrail, constraint, safety, retention, privacy, identity, vault, hive, swarm, etc.
- Do NOT promote:
  - Implementation plans, specs, designs, migration guides
  - Build/test results, crash reports, fix summaries
  - Audit reports, gap analyses, deployment runbooks
  - `SUPERTONIC_2_*` files
- When in doubt, leave it out and flag for human review.

---

## Naming and Path Conventions
- Use repo-relative paths everywhere: `EVO/smartdocs/raw/...`
- Never use `/Users/lsctech/...` absolute paths
- Filenames use em-dash (`—`), not en-dash (`–`). If a source uses en-dash, rename to em-dash on mirror.

---

## Index Maintenance
Both `lsctech-wiki/EVO/wiki/doctrine/index.md` and `EVO-wiki/wiki/doctrine/index.md` are pointer tables, not doctrine content.
- When promoting new doctrine, append a row to the table in **both** index files.
- Do not edit existing index rows unless the source path is stale.
- Keep `index.md` out of the doctrine promotion workflow; it is infrastructure.

---

## Infrastructure Files in `wiki/doctrine/`
The following files are allowed in `wiki/doctrine/` but are NOT doctrine content:
- `index.md`
- `_lifecycle-manifest.md` (only if present; prefer removing it to `wiki/admin/` if that folder exists)

No other infrastructure or manifest files should land in `wiki/doctrine/`.

---

## Review Checklist
Before marking any promotion complete:
- [ ] Both wikis contain identical files and content
- [ ] Every file has OKF frontmatter, `## Related`, and `^[...]` provenance
- [ ] No absolute `/Users/lsctech/...` paths remain
- [ ] No archive markers or "Raw Draft" titles remain
- [ ] Index tables updated in both wikis
- [ ] Sources mirrored to `wiki/source-materials/mirrors/doctrine/`

---

## Drift Recovery
If the two wikis diverge:
1. Treat `EVO-wiki` as the canonical reference unless told otherwise.
2. Copy the newer/changed file from `EVO-wiki` to `lsctech-wiki`.
3. Re-run the review checklist.
