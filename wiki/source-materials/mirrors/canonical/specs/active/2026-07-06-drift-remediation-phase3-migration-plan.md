---
title: "Legacy Migration (Phase 3) Implementation Plan"
type: spec
tags: ['lsctech', 'spec', 'source-material', 'canonical', 'evo']
updated: 2026-07-06
---

# Legacy Migration (Phase 3) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Migrate any authoritative content out of the legacy `docs/` tree into
`smartdocs/`, fix every reference to `docs/` elsewhere in the repo, then delete
`docs/` entirely — making `smartdocs/` the repository's sole documentation root, per
`smartdocs/specs/raw/2026-07-05-smartdocs-drift-remediation.md` Section 1.

**Architecture:** Five content-disposition/reference-fix tasks that can run in any
order, followed by one final deletion+verification task that depends on all five
completing. Content dispositions are evidence-based (per-file diff against
`smartdocs/`, not assumption) and every disposition — migrate or discard — is
recorded in a decision log, never silently dropped.

**Tech Stack:** Markdown, JSON, git, `grep`/`diff` for content comparison.

## Global Constraints

- **Do not run `polaris adopt consolidate`.** Its pre-generated plan
  (`.polaris/adoption-plan.json`) has a confirmed destination-collision bug for this
  exact migration (all `docs/EVOnotes/*/README.md` move steps map to the identical
  `smartdocs/raw/README.md`) — running it would silently overwrite that file 29 times.
  All moves in this plan are manual, per-file, reviewed moves.
- Repo-root `specs/` (distinct from `smartdocs/specs/`) is explicitly **out of
  scope** — a separate spec-kit feature-spec system, not part of this migration.
- Every content disposition (migrate or discard) must be recorded with its reasoning
  in `.superpowers/sdd/phase3-decision-log.md` (created by Task 2, appended to by
  Tasks 2-4) — no file gets silently deleted without a recorded reason.
- Do not delete `docs/` (Task 6) until Tasks 1-5 are all complete and committed.
- The audit file `.superpowers/sdd/phase3-docs-audit-2026-07-06.md` contains the full
  research this plan is based on — read it as context, but re-verify specific claims
  with the exact commands each task specifies rather than trusting it blindly (it's
  from a fast similarity-grep pass, not per-file diffs).

---

### Task 1: Migrate `docs/architecture/alice-hosting/` to `smartdocs/doctrine/active/`

**Correction (2026-07-06):** an earlier draft of this plan targeted
`smartdocs/architecture/` — that bucket does **not exist** in this repo (git-fit's
`smartdocs/` top level is only `audits/`, `decisions/`, `doctrine/`, `raw/`,
`runtime/`, `specs/` — confirmed via `ls smartdocs/`). This was a conflation with a
*different* repo's smartdocs structure (the `polaris-clo-smartdocs` mirror, which does
have an `architecture/` bucket). The corrected, evidenced destination is
`smartdocs/doctrine/active/` — confirmed by `smartdocs/doctrine/active/polaris-run-state-architecture.md`
already existing there as a precedent for architecture-flavored content in this
specific repo's `doctrine/active/` bucket.

**Files:**
- Move: `docs/architecture/alice-hosting/anchor-runtime.md` → `smartdocs/doctrine/active/alice-hosting-anchor-runtime.md`
- Move: `docs/architecture/alice-hosting/privileged-broker.md` → `smartdocs/doctrine/active/alice-hosting-privileged-broker.md`
- Move: `docs/architecture/alice-hosting/user-profile-isolation.md` → `smartdocs/doctrine/active/alice-hosting-user-profile-isolation.md`
- Move: `docs/architecture/alice-hosting/governance-map.md` → `smartdocs/doctrine/active/alice-hosting-governance-map.md`
- Move: `docs/architecture/alice-hosting/v1-strategy.md` → `smartdocs/doctrine/active/alice-hosting-v1-strategy.md`
- Create: `.superpowers/sdd/phase3-decision-log.md`

**Interfaces:** None — independent of Tasks 2-5. Task 6 depends on this completing.

- [ ] **Step 1: Confirm these 5 files are still present and still absent from smartdocs/, and re-confirm smartdocs/'s actual top-level structure**

Run:
```bash
cd /Users/lsctech/Developer/git-fit
ls docs/architecture/alice-hosting/
ls smartdocs/
find smartdocs -iname "*alice-hosting*" -o -iname "*anchor-runtime*" -o -iname "*privileged-broker*"
```
Expected: first command lists the 5 `.md` files; second command lists exactly
`audits`, `decisions`, `doctrine`, `raw`, `runtime`, `specs` (no `architecture`); third
command returns no output (confirms no existing equivalent). If `smartdocs/` now has
an `architecture/` directory that didn't exist before, stop and reconsider the
destination — the plan's reasoning assumed it doesn't exist.

- [ ] **Step 2: Create the decision log with its header and this task's entries**

Create `.superpowers/sdd/phase3-decision-log.md`:

```markdown
# Phase 3 Migration Decision Log

Every disposition (migrate or discard) for content under legacy `docs/`, with
reasoning. Appended to by each Phase 3 task — never edit past entries, only add.

## Task 1: docs/architecture/alice-hosting/

| File | Disposition | Reasoning |
|---|---|---|
| anchor-runtime.md | Migrated to smartdocs/doctrine/active/alice-hosting-anchor-runtime.md | Confirmed authoritative (cluster EVOC-322, 2026-06-30), no existing smartdocs/ equivalent; smartdocs/architecture/ doesn't exist in this repo, doctrine/active/ is the evidenced convention for architecture-flavored content here |
| privileged-broker.md | Migrated to smartdocs/doctrine/active/alice-hosting-privileged-broker.md | Same as above |
| user-profile-isolation.md | Migrated to smartdocs/doctrine/active/alice-hosting-user-profile-isolation.md | Same as above |
| governance-map.md | Migrated to smartdocs/doctrine/active/alice-hosting-governance-map.md | Same as above |
| v1-strategy.md | Migrated to smartdocs/doctrine/active/alice-hosting-v1-strategy.md | Same as above |
```

- [ ] **Step 3: Move all 5 files, flattening the subdirectory into the filename prefix**

```bash
cd /Users/lsctech/Developer/git-fit
git mv docs/architecture/alice-hosting/anchor-runtime.md smartdocs/doctrine/active/alice-hosting-anchor-runtime.md
git mv docs/architecture/alice-hosting/privileged-broker.md smartdocs/doctrine/active/alice-hosting-privileged-broker.md
git mv docs/architecture/alice-hosting/user-profile-isolation.md smartdocs/doctrine/active/alice-hosting-user-profile-isolation.md
git mv docs/architecture/alice-hosting/governance-map.md smartdocs/doctrine/active/alice-hosting-governance-map.md
git mv docs/architecture/alice-hosting/v1-strategy.md smartdocs/doctrine/active/alice-hosting-v1-strategy.md
```

- [ ] **Step 4: Verify the move and that the now-empty source directory is gone**

```bash
cd /Users/lsctech/Developer/git-fit
ls smartdocs/doctrine/active/ | grep alice-hosting
find docs/architecture -type f
```
Expected: first command lists all 5 new filenames; second command returns no output
(the `alice-hosting/` subdirectory should be empty and git doesn't track empty dirs —
if `docs/architecture/` itself still exists as an empty dir, that's fine, it'll be
removed in Task 6's full `docs/` deletion).

- [ ] **Step 5: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add smartdocs/doctrine/active/ docs/architecture/ .superpowers/sdd/phase3-decision-log.md
git commit -m "$(cat <<'EOF'
Migrate docs/architecture/alice-hosting/ to smartdocs/doctrine/active/

5 architecture-decision docs from cluster EVOC-322 (2026-06-30) had
no equivalent anywhere in smartdocs/ — confirmed authoritative,
unmigrated content per the 2026-07-05 drift-remediation spec. Target
is doctrine/active/, not a separate architecture/ bucket — this repo's
smartdocs/ has no architecture/ directory (unlike the polaris-clo-
smartdocs mirror, a different repo); doctrine/active/ is the
evidenced convention here, per polaris-run-state-architecture.md
already living there. Flattened the alice-hosting/ subdirectory into
filename prefixes. Starts the Phase 3 decision log that subsequent
tasks append to.
EOF
)"
```

---

### Task 2: Audit and disposition `docs/EVOnotes/` (26 files)

**Files:**
- Read: all 26 files under `docs/EVOnotes/` (list in Step 1)
- Modify: `.superpowers/sdd/phase3-decision-log.md` (append)
- Possibly create/move files under `smartdocs/` — only if Step 3's diff finds
  genuinely unique content (see below; based on the 2026-07-06 audit, expect this to
  be rare, but do not assume it without checking)

**Interfaces:** Consumes the decision-log file created in Task 1 (append to it, don't
recreate it). Independent of Tasks 1, 3, 4, 5 otherwise. Task 6 depends on this
completing.

**Context:** Read `.superpowers/sdd/phase3-docs-audit-2026-07-06.md`'s "docs/EVOnotes/"
section first — it lists all 26 files with bucket and content summary from a prior
research pass. Treat it as a starting hypothesis (25 of 26 are lifecycle/domain index
READMEs, not primary content; the doctrine they describe appears to already exist in
`smartdocs/doctrine/active/`) — verify each file's disposition yourself per the steps
below rather than trusting the audit file's characterization directly.

- [ ] **Step 1: List the 26 files to confirm the count and set is unchanged**

```bash
cd /Users/lsctech/Developer/git-fit
find docs/EVOnotes -type f | sort
```
Expected: 26 lines (25 `.md` files + `EVOnotes.base`). If the count differs from 26,
stop and reconcile with the audit file before proceeding — something changed.

- [ ] **Step 2: For each of the 25 index/README files, determine whether it is a pure
  navigation aid (lists/describes other files) or contains primary content of its own**

For each file, read it, then classify: does the file assert doctrine/facts about the
system itself (primary content), or does it only describe/list other files
(navigation aid)? Per the audit file, 24 of 25 markdown files are navigation aids;
`needs-review/ai/Claude.md` is flagged as having real prose content ("Claude
Instructions for EVO"). Confirm this classification yourself for at least
`needs-review/ai/Claude.md` and spot-check 3 others.

- [ ] **Step 3: For `needs-review/ai/Claude.md` specifically — resolve its "superseded
  by docs/agents/Claude.md" claim**

```bash
cd /Users/lsctech/Developer/git-fit
cat docs/EVOnotes/needs-review/ai/Claude.md
ls docs/agents/ 2>&1   # expect: No such file or directory (confirmed dangling)
grep -rl "Claude Instructions" --include="*.md" smartdocs/ CLAUDE.md AGENTS.md 2>/dev/null
```
Determine: is there a CURRENT canonical replacement for this content anywhere in
`smartdocs/` or the root `CLAUDE.md`/`AGENTS.md`? (Root `CLAUDE.md`/`AGENTS.md` are
thin pointer files to `POLARIS_RULES.md`, not detailed instruction content — so if the
grep finds nothing in `smartdocs/`, this content may be genuinely orphaned, not just
duplicated.) Record your finding in the decision log — if no replacement exists and
the content still describes real, current agent behavior, migrate it to
`smartdocs/doctrine/candidate/` (let the normal review pipeline from Phase 2 handle
promoting or deprecating it) rather than discarding it. If it's confirmed
superseded by content that does exist, record where and discard.

- [ ] **Step 4: For the remaining 24 navigation-aid files, confirm no unique
  organizational information is lost by checking whether an equivalent index exists
  anywhere in `smartdocs/`**

```bash
cd /Users/lsctech/Developer/git-fit
find smartdocs -iname "index.md" -o -iname "SUMMARY.md"
```
Read a sample of what these existing index files cover. If `smartdocs/` already has
its own navigation/index convention (it does — `index.md` files per directory, per
the polaris-clo-smartdocs mirror's convention referenced in this repo's own
`smartdocs/index.md` if one exists, or `SUMMARY.md` files), then `docs/EVOnotes/`'s
bespoke lifecycle-stage/domain-traversal structure is a superseded, parallel
navigation scheme — record each of the 24 files as "superseded, no unique content,
smartdocs/ uses its own index convention" in the decision log, rather than trying to
recreate the old structure.

- [ ] **Step 5: Append all 26 dispositions to the decision log**

Append to `.superpowers/sdd/phase3-decision-log.md`:

```markdown

## Task 2: docs/EVOnotes/ (26 files)

| File | Disposition | Reasoning |
|---|---|---|
```

Fill in one row per file with the actual disposition and reasoning determined in
Steps 2-4 (do not leave this as a template — every one of the 26 files needs its own
row with real content).

- [ ] **Step 6: Execute the dispositions**

For files disposed as "discard/superseded": no action needed yet (Task 6 deletes all
of `docs/` at once — don't delete individual files here, just record the decision).

For `needs-review/ai/Claude.md` if disposed as "migrate": 

```bash
cd /Users/lsctech/Developer/git-fit
git mv docs/EVOnotes/needs-review/ai/Claude.md smartdocs/doctrine/candidate/claude-instructions-for-evo.md
```

(Adjust the destination filename only if Step 3 determined a different disposition —
the command above is for the "migrate to candidate for review" outcome specifically.)

- [ ] **Step 7: Verify**

```bash
cd /Users/lsctech/Developer/git-fit
grep -c "^|" .superpowers/sdd/phase3-decision-log.md
```
Expected: a count reflecting Task 1's 5 rows (+2 header/separator) plus Task 2's 26
rows (+2 header/separator) = at least 35 total pipe-lines. If any migration happened,
confirm the destination file exists.

- [ ] **Step 8: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add .superpowers/sdd/phase3-decision-log.md smartdocs/
git commit -m "$(cat <<'EOF'
Audit and disposition docs/EVOnotes/ (26 files)

Per-file review: 24 of 25 markdown files are lifecycle/domain
navigation aids describing content that already exists in
smartdocs/doctrine/active/ under smartdocs/'s own index convention —
recorded as superseded with no unique content. needs-review/ai/
Claude.md contained real content ("Claude Instructions for EVO")
self-described as superseded by a nonexistent docs/agents/Claude.md
path; resolved per the decision log. Full reasoning per file in
.superpowers/sdd/phase3-decision-log.md.
EOF
)"
```

---

### Task 3: Audit and disposition `docs/raw/` (all subdirectories)

**Files:**
- Read: `docs/raw/README.md`, `docs/raw/archived/*`, `docs/raw/audits-and-issues/README.md`,
  `docs/raw/contracts/README.md`, `docs/raw/contracts/alice/*.json`,
  `docs/raw/deprecated/README.md`
- Modify: `.superpowers/sdd/phase3-decision-log.md` (append)
- Possibly move: `docs/raw/contracts/alice/alice_manifest.example.json` and
  `alice_manifest.schema.json` (see Step 3)

**Interfaces:** Consumes the decision-log file (append). Independent of Tasks 1, 2, 4,
5. Task 6 depends on this completing.

**Context:** Read `.superpowers/sdd/phase3-docs-audit-2026-07-06.md`'s "docs/raw/"
section first.

- [ ] **Step 1: Confirm the current file/subdirectory counts**

```bash
cd /Users/lsctech/Developer/git-fit
find docs/raw -type f | wc -l
find docs/raw -maxdepth 1 -type d
```
Expected: consistent with the audit (README.md + archived/ [98 files per its own
README] + audits-and-issues/ [66 Linear exports] + contracts/ [schema+example+README]
+ deprecated/ [24 files]).

- [ ] **Step 2: Disposition `archived/`, `audits-and-issues/`, and `deprecated/` as bulk buckets**

These three subdirectories are explicitly self-described (by their own README.md
files, already read in the prior audit) as non-canonical: `archived/` is uncanonized
Notion exports/duplicates, `audits-and-issues/` is Linear-issue records ("not
doctrine"), `deprecated/` is stale-reference rejections for systems already
confirmed gone (MLX, ElevenLabs TTS, ENF/VOICE adapters, Phi-4-mini — consistent with
`smartdocs/doctrine/deprecated/`'s own anti-drift notes elsewhere in this repo). Spot
check 5 files across these three directories (not all ~188) to confirm none contain
something surprising that contradicts their self-description:

```bash
cd /Users/lsctech/Developer/git-fit
find docs/raw/archived docs/raw/audits-and-issues docs/raw/deprecated -type f | shuf -n 5 --random-source=<(yes 42)
```
Read the 5 sampled files. If all confirm the bucket's self-description, record the
whole bucket as one decision-log entry each (not 188 individual rows). If any
surprise you (e.g. a file that looks like live doctrine, not an export/deprecation
record), read it fully and disposition it individually instead.

- [ ] **Step 3: Disposition `contracts/alice/` (real, substantive content — not a stub)**

```bash
cd /Users/lsctech/Developer/git-fit
grep -rl "alice_manifest" smartdocs/ 2>/dev/null
find smartdocs -iname "*alice*manifest*" -o -iname "*avatar*"
```
Expected: likely no output (per the 2026-07-05 audit, this schema/example pair
appeared nowhere in `smartdocs/`). If confirmed absent, this is real, substantive,
unmigrated content (an active JSON Schema + example for the Alice avatar runtime
manifest, referenced by `flutter_app/assets/alice/alice_manifest.json` per the
schema's own metadata) — migrate it. JSON schemas don't fit `smartdocs/`'s
markdown-oriented buckets; the most consistent home is alongside where the format is
actually used, so:

```bash
cd /Users/lsctech/Developer/git-fit
mkdir -p smartdocs/doctrine/active/alice-manifest
git mv docs/raw/contracts/alice/alice_manifest.example.json smartdocs/doctrine/active/alice-manifest/alice_manifest.example.json
git mv docs/raw/contracts/alice/alice_manifest.schema.json smartdocs/doctrine/active/alice-manifest/alice_manifest.schema.json
git mv docs/raw/contracts/README.md smartdocs/doctrine/active/alice-manifest/README.md
```

- [ ] **Step 4: Append dispositions to the decision log**

```markdown

## Task 3: docs/raw/ (9 top-level entries)

| Entry | Disposition | Reasoning |
|---|---|---|
| README.md | Discard | Governance doc for the raw/ staging area itself; smartdocs/raw/ has its own equivalent convention already in use |
| archived/ (98 files) | Discard, bulk | Self-described uncanonized Notion exports/duplicates; spot-check confirmed |
| audits-and-issues/ (66 files) | Discard, bulk | Self-described Linear-issue records, not doctrine; spot-check confirmed |
| deprecated/ (24 files) | Discard, bulk | Self-described stale-reference rejections for already-confirmed-gone systems; spot-check confirmed |
| contracts/alice/ (2 JSON files + README) | Migrated to smartdocs/doctrine/active/alice-manifest/ | Real, substantive, unmigrated content — confirmed no smartdocs/ equivalent |
```

(Adjust rows if Step 2's spot-check found a surprise requiring individual disposition.)

- [ ] **Step 5: Verify**

```bash
cd /Users/lsctech/Developer/git-fit
ls smartdocs/doctrine/active/alice-manifest/
python3 -m json.tool smartdocs/doctrine/active/alice-manifest/alice_manifest.schema.json > /dev/null && echo "schema JSON valid"
python3 -m json.tool smartdocs/doctrine/active/alice-manifest/alice_manifest.example.json > /dev/null && echo "example JSON valid"
```
Expected: 3 files listed, both "valid" messages printed.

- [ ] **Step 6: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add smartdocs/doctrine/active/alice-manifest/ .superpowers/sdd/phase3-decision-log.md
git commit -m "$(cat <<'EOF'
Audit and disposition docs/raw/ (9 top-level entries)

archived/ (98 files), audits-and-issues/ (66 files), and deprecated/
(24 files) confirmed non-canonical by their own self-descriptions and
spot-checking — no migration needed. contracts/alice/ (JSON Schema +
example for the Alice avatar runtime manifest) was real, substantive,
unmigrated content with no smartdocs/ equivalent — migrated to
smartdocs/doctrine/active/alice-manifest/. Full reasoning in
.superpowers/sdd/phase3-decision-log.md.
EOF
)"
```

---

### Task 4: Disposition `docs/EVO Book Draft/`

**Files:**
- Read: `docs/EVO Book Draft/EVO Book Draft.base`
- Modify: `.superpowers/sdd/phase3-decision-log.md` (append)

**Interfaces:** Consumes the decision-log file (append). Independent of Tasks 1, 2, 3,
5. Task 6 depends on this completing.

- [ ] **Step 1: Confirm whether any actual chapter content exists under this directory
  (the audit only found the `.base` manifest, not chapter files)**

```bash
cd /Users/lsctech/Developer/git-fit
find "docs/EVO Book Draft" -type f
```
Expected: per the audit, only `EVO Book Draft.base`. If this turns up additional
files (actual chapter markdown), read them and expand this task's disposition to
cover them individually — don't just handle the `.base` file if there's more here.

- [ ] **Step 2: Read the `.base` file and confirm it's purely a view manifest, not content**

```bash
cd /Users/lsctech/Developer/git-fit
cat "docs/EVO Book Draft/EVO Book Draft.base"
```
Confirm it's an Obsidian table-view config (columns like Part, Summary, Chapter Type,
Protected, Status) with no actual prose. This is not doctrine and has no smartdocs/
equivalent to check against — it's creative/book-writing tooling, categorically
different from anything else in this migration.

- [ ] **Step 3: Append the disposition to the decision log**

```markdown

## Task 4: docs/EVO Book Draft/

| Entry | Disposition | Reasoning |
|---|---|---|
| EVO Book Draft.base | Discard | Obsidian view-manifest for book-writing tooling, no prose content, not doctrine, no smartdocs/ equivalent needed — categorically outside the documentation system this migration concerns |
```

(If Step 1 found actual chapter files, add rows for each with their own disposition —
don't discard prose content without reading it first.)

- [ ] **Step 4: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add .superpowers/sdd/phase3-decision-log.md
git commit -m "$(cat <<'EOF'
Disposition docs/EVO Book Draft/

Confirmed the directory contains only an Obsidian view-manifest
(.base file) with no prose content — book-writing tooling, not
doctrine, categorically outside this migration's scope. No smartdocs/
migration needed. Recorded in the Phase 3 decision log.
EOF
)"
```

---

### Task 5: Fix all references to `docs/` outside the tree itself

**Files:**
- Modify: `.polaris/skills/polaris-analyze/SKILL.md`
- Modify: `.polaris/skills/polaris-analyze/steps/01-fetch-and-orient.md`
- Modify: `.polaris/skills/polaris-analyze/steps/03-assess-issue.md`
- Modify: `.polaris/skills/polaris-analyze/steps/05-create-cluster-plan.md`
- Modify: `.polaris/skills/polaris-analyze/linked-skills/repo-analysis.md`
- Modify: `.polaris/skills/polaris-run/SKILL.md`
- Modify: `.polaris/skills/docs-ingest/chain.json`
- Modify: `.evo/run-state/README.md`
- Modify: `.claude/settings.local.json`
- Modify or retire: `.github/spec-kit/scripts/bash/classify-raw-docs.sh`

**Interfaces:** None — independent of Tasks 1-4. Task 6 depends on this completing
(its success-criteria grep will fail if any of these aren't fixed first).

- [ ] **Step 1: Fix the two dangling `docs/Polaris/spec/...` references**

Read each file first to find the exact line, then replace the dangling path with the
correct, existing equivalent. Run this to find every occurrence:

```bash
cd /Users/lsctech/Developer/git-fit
grep -rn "docs/Polaris/spec" .polaris/skills/
```

**Correction (2026-07-06):** an earlier draft of this step proposed
`smartdocs/architecture/polaris-run-ledger.md` and `smartdocs/integrations/index.md`
as replacement destinations — neither file exists (verified via `ls`); this repo's
`smartdocs/` has no `architecture/` or `integrations/` directory at all (conflated
with the different `polaris-clo-smartdocs` mirror repo again). Do not use those paths.
Instead, for each match found (expected in
`.polaris/skills/polaris-analyze/SKILL.md`, `.polaris/skills/polaris-run/SKILL.md`,
`.polaris/skills/polaris-analyze/steps/01-fetch-and-orient.md`,
`.polaris/skills/polaris-analyze/steps/03-assess-issue.md`,
`.polaris/skills/polaris-analyze/steps/05-create-cluster-plan.md`,
`.polaris/skills/polaris-analyze/linked-skills/repo-analysis.md`):

For `docs/Polaris/spec/polaris-implementation-plan.md`, first read both of these real
candidate files to judge which one the citing context actually means (both exist,
confirmed via `ls`):
```bash
cd /Users/lsctech/Developer/git-fit
head -30 smartdocs/doctrine/active/polaris-run-state-architecture.md
head -30 smartdocs/runtime/run-reports/polaris-architecture-spec.md
```
`polaris-run-state-architecture.md` is framed as an "EVO Precursor Planning Spec...
Future Polaris Extraction Candidate" (about run-state/telemetry design).
`polaris-architecture-spec.md` is framed as a "Draft — analysis artifact for
EVOS1-373" (an analysis document, `status: raw`, itself citing `.evo/routing.md` —
another dangling reference, out of scope to fix here but worth noting if you spot
it). Read the actual citing sentence in each of the 6 files above ("See `docs/Polaris/
spec/polaris-implementation-plan.md` for the Polaris architecture reference") and
judge which candidate matches "the Polaris architecture reference" better in context.
If genuinely unclear, prefer `smartdocs/doctrine/active/polaris-run-state-architecture.md`
(active/stable status beats a `raw` draft analysis artifact for a "reference" citation)
but note the ambiguity in your report.

For `docs/Polaris/spec/provider-capabilities.md`: run
`grep -rl "provider.capabilit" smartdocs/ 2>/dev/null` — if this returns nothing (no
clean equivalent exists, unlike the run-state case), do not invent a destination.
Instead replace the citation with a note that no `smartdocs/` equivalent currently
exists yet, e.g. change "See `docs/Polaris/spec/provider-capabilities.md` for the full
provider model." to "No current `smartdocs/` doc covers the full provider model in
detail — this reference is stale and needs a real replacement written or cited once
one exists." Do not point at a file that doesn't exist, and do not silently delete the
sentence — flag the gap.

- [ ] **Step 2: Fix `.evo/run-state/README.md`'s dangling reference**

```bash
cd /Users/lsctech/Developer/git-fit
cat .evo/run-state/README.md
```
This file cites `docs/EVOnotes/planning-specs/polaris-run-state-architecture.md`
(confirmed nonexistent). A real, well-matched replacement already exists — verify it
yourself:
```bash
cd /Users/lsctech/Developer/git-fit
head -15 smartdocs/doctrine/active/polaris-run-state-architecture.md
```
Expected: frontmatter shows `source: smartdocs/raw/polaris-run-state-architecture.md`
and a body titled "EVO Precursor Planning Spec / Run-State Architecture And
Tracker-Agnostic Execution Telemetry / Future Polaris Extraction Candidate" — this
matches `.evo/run-state/README.md`'s own description of itself ("the Polaris
precursor pattern"). Replace the citation with
`smartdocs/doctrine/active/polaris-run-state-architecture.md`.

- [ ] **Step 3: Update the two live (not-yet-broken) `docs/` references in polaris-analyze**

In `.polaris/skills/polaris-analyze/SKILL.md`, find:
```
- Create implementation plans and specs (in `docs/`)
```
Replace with:
```
- Create implementation plans and specs (in `smartdocs/specs/raw/`)
```

In `.polaris/skills/polaris-analyze/steps/05-create-cluster-plan.md`, find:
```
  - docs/ (for any analysis docs produced)
```
Replace with:
```
  - smartdocs/specs/raw/ (for any analysis docs produced)
```

- [ ] **Step 4: Update `docs-ingest/chain.json`'s forbidden-action wording**

Read `.polaris/skills/docs-ingest/chain.json`. Find the forbidden action
`"root docs/ writes for new Smart Docs"`. Since `docs/` will no longer exist after
Task 6, update the wording to name `smartdocs/` explicitly for clarity going forward:
change it to `"writes outside smartdocs/ for new Smart Docs"`.

Run `python3 -m json.tool .polaris/skills/docs-ingest/chain.json > /dev/null` after
editing to confirm it's still valid JSON.

- [ ] **Step 5: Remove the `docs/` permission entry from `.claude/settings.local.json`**

Read `.claude/settings.local.json`, find the line `"Bash(git add docs/*)"`, and remove
it (it will be meaningless once `docs/` no longer exists). Verify the file is still
valid JSON after editing: `python3 -m json.tool .claude/settings.local.json > /dev/null`.

- [ ] **Step 6: Retire `.github/spec-kit/scripts/bash/classify-raw-docs.sh`**

```bash
cd /Users/lsctech/Developer/git-fit
grep -rl "classify-raw-docs" --include="*.md" --include="*.yml" --include="*.json" . --exclude-dir=node_modules --exclude-dir=.git
```
This script's entire purpose (per its own header comment) is classifying
`docs/raw/` — which will no longer exist. Check the grep results: if
`docs/raw/README.md` is the only thing that references it (and that file is being
deleted in Task 6 anyway), delete the script:

```bash
cd /Users/lsctech/Developer/git-fit
git rm .github/spec-kit/scripts/bash/classify-raw-docs.sh
```

If anything else references it, stop and report rather than deleting a script something
else depends on.

- [ ] **Step 7: Verify all fixes**

```bash
cd /Users/lsctech/Developer/git-fit
grep -rn "docs/Polaris/spec" .polaris/skills/ 2>/dev/null
grep -n "polaris-run-state-architecture" .evo/run-state/README.md
grep -n "docs/" .polaris/skills/polaris-analyze/SKILL.md .polaris/skills/polaris-analyze/steps/05-create-cluster-plan.md
grep -n "docs/" .claude/settings.local.json
python3 -m json.tool .polaris/skills/docs-ingest/chain.json > /dev/null && echo "docs-ingest chain.json still valid"
python3 -m json.tool .claude/settings.local.json > /dev/null && echo "settings.local.json still valid"
ls .github/spec-kit/scripts/bash/classify-raw-docs.sh 2>&1
```
Expected: first grep returns nothing; second grep returns nothing (or shows the
corrected reference, not the dangling one); third and fourth greps show only
`smartdocs/`-pointing lines; both "still valid" messages print; last command reports
"No such file or directory".

- [ ] **Step 8: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add .polaris/skills/polaris-analyze/ .polaris/skills/polaris-run/SKILL.md .polaris/skills/docs-ingest/chain.json .evo/run-state/README.md .claude/settings.local.json .github/spec-kit/scripts/bash/classify-raw-docs.sh
git commit -m "$(cat <<'EOF'
Fix all remaining docs/ references ahead of its deletion

Two confirmed dangling docs/Polaris/spec/* references in
polaris-analyze/polaris-run (6 files) redirected to real smartdocs/
equivalents. .evo/run-state/README.md's dangling reference to a
nonexistent docs/EVOnotes/planning-specs file corrected. Two live
polaris-analyze references updated from docs/ to smartdocs/specs/raw/.
docs-ingest/chain.json's forbidden-action wording updated to name
smartdocs/ explicitly. Removed the now-meaningless docs/ permission
entry from .claude/settings.local.json. Retired
classify-raw-docs.sh, whose entire purpose was classifying
docs/raw/, which no longer exists after this migration.
EOF
)"
```

---

### Task 6: Delete `docs/` and run final verification

**Files:**
- Delete: `docs/` (entire tree)

**Interfaces:** Consumes: all of Tasks 1-5 must be complete and committed first — this
task's own Step 1 verifies that precondition before deleting anything.

- [ ] **Step 1: Verify Tasks 1-5 are complete before deleting anything**

```bash
cd /Users/lsctech/Developer/git-fit
grep -c "^##" .superpowers/sdd/phase3-decision-log.md
find docs/architecture/alice-hosting -type f 2>&1
find docs/raw/contracts/alice -type f 2>&1
grep -rn "docs/Polaris/spec" .polaris/skills/ 2>/dev/null
```
Expected: first command shows 4 (Tasks 1-4 each added one `##` section); second and
third commands report "No such file or directory" (confirms Tasks 1 and 3's moves
happened); fourth command returns nothing (confirms Task 5's fixes happened). If any
of these don't match, STOP — do not proceed to deletion until all prior tasks are
verifiably complete.

- [ ] **Step 2: Delete the entire `docs/` tree**

```bash
cd /Users/lsctech/Developer/git-fit
git rm -r docs/
```

- [ ] **Step 3: Run the Section 4 success-criteria grep from the parent spec**

```bash
cd /Users/lsctech/Developer/git-fit
grep -rn "docs/" --include="*.md" --include="*.json" --include="*.ts" --include="*.js" --include="*.sh" \
  --exclude-dir=node_modules --exclude-dir=.git --exclude-dir=smartdocs --exclude-dir=.svelte-kit \
  --exclude-dir=dist --exclude-dir=build --exclude-dir=specs . | grep -v "\.taskchain_artifacts\|\.polaris/clusters\|\.polaris/adoption"
```
(Historical run artifacts under `.taskchain_artifacts/` and `.polaris/clusters/*/packets/`
are immutable records of already-completed runs — per Section 2.C of the parent spec,
these are intentionally not rewritten. `.polaris/adoption-inventory.json`/
`adoption-plan.json` are themselves scan/plan output referencing the old state at scan
time — also not rewritten, since they're historical scan records, not live config.
Repo-root `specs/` is explicitly out of scope per this plan's Global Constraints.)

Expected: no output, or only matches inside the excluded historical/out-of-scope
paths. If anything else turns up, stop and fix it before finalizing — don't leave a
dangling reference behind just because deletion already happened.

- [ ] **Step 4: Verify `docs/` is fully gone**

```bash
cd /Users/lsctech/Developer/git-fit
ls docs 2>&1
git status --short | head -5
```
Expected: "No such file or directory"; `git status --short` shows the deletion staged.

- [ ] **Step 5: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git commit -m "$(cat <<'EOF'
Delete docs/ — smartdocs/ is now the sole documentation root

All authoritative content audited and dispositioned (Tasks 1-4,
recorded in .superpowers/sdd/phase3-decision-log.md): the 5
alice-hosting architecture docs and the Alice manifest JSON
Schema/example migrated to smartdocs/; docs/EVOnotes/'s 26 files
(mostly navigation aids for content already in smartdocs/) and
docs/raw/'s ~188 non-canonical files (exports, issue records,
deprecation records) confirmed superseded with nothing unique lost;
docs/EVO Book Draft/ confirmed to contain no prose content. All
references elsewhere in the repo fixed (Task 5). Final grep confirms
no remaining docs/ references outside historical/immutable run
artifacts and the explicitly out-of-scope repo-root specs/.

Completes the target architecture from
smartdocs/specs/raw/2026-07-05-smartdocs-drift-remediation.md
Section 1.
EOF
)"
```

---

## Verification (whole plan)

- [ ] **Final check: docs/ gone, decision log complete, no dangling references**

```bash
cd /Users/lsctech/Developer/git-fit
ls docs 2>&1                                          # expect: No such file or directory
wc -l .superpowers/sdd/phase3-decision-log.md          # expect: substantial, all 4 task sections present
grep -c "^##" .superpowers/sdd/phase3-decision-log.md  # expect: 4
ls smartdocs/doctrine/active/ | grep -c alice-hosting     # expect: 5
ls smartdocs/doctrine/active/alice-manifest/              # expect: 3 files
grep -rn "docs/Polaris/spec" .polaris/skills/ 2>/dev/null  # expect: nothing
```

Expected: all checks pass as annotated.