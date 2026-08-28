---
title: "Repo Hygiene (Phase 1 + 4) Implementation Plan"
type: spec
tags: ['lsctech', 'spec', 'source-material', 'canonical', 'evo']
updated: 2026-07-05
---

# Repo Hygiene (Phase 1 + 4) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix the confirmed repo-hygiene issues in git-fit — a tracked regenerable
binary, a stray artifact file, an untracked junk directory — and produce the read-only
branch inventory (Phase 4 of the parent spec). Bundled here because all four tasks are
independent, low-risk, and need no approval gate, unlike Phases 2/3/5. Does not touch
smartdocs governance or the docs/ migration.

**Architecture:** Four independent git-level fixes, each its own commit, verified by
`git status`/`git check-ignore` rather than a test suite (this is repo hygiene, not
application code). No step depends on any other step in this plan.

**Tech Stack:** git, bash.

## Global Constraints

- Every git operation is a normal commit — no `--force`, no history rewriting, no
  branch deletion (branch inventory in this plan is read-only reporting only, per
  `smartdocs/specs/raw/2026-07-05-smartdocs-drift-remediation.md` Section 6, Phase 4).
- Work happens on the current branch (`evos1-387-delivery`) unless the executing agent
  is told otherwise at run time.
- Do not touch anything under `docs/`, `smartdocs/doctrine/`, or `smartdocs/raw/` —
  that's Phase 2/3 scope, tracked in separate plans.

---

### Task 1: Untrack the regenerable graph database

**Files:**
- Modify: `.gitignore` (append `*.sqlite`)
- Remove from git index: `.polaris/graph/graph.sqlite` (file stays on disk, untracked)

**Interfaces:** None — this task has no dependents in this plan.

- [ ] **Step 1: Confirm current state**

Run: `cd /Users/lsctech/Developer/git-fit && git ls-files .polaris/graph/graph.sqlite`
Expected: prints `.polaris/graph/graph.sqlite` (confirms it's tracked before the fix)

- [ ] **Step 2: Confirm `.polarisignore` already excludes sqlite (context, not a dependency)**

Run: `grep -n sqlite /Users/lsctech/Developer/git-fit/.polarisignore`
Expected: prints `*.sqlite` — confirms Polaris's own tooling already treats this file
as noise; git tracking it is the only inconsistency being fixed.

- [ ] **Step 3: Add `*.sqlite` to `.gitignore`**

Append to `/Users/lsctech/Developer/git-fit/.gitignore`:

```
*.sqlite
```

- [ ] **Step 4: Untrack the file without deleting it**

Run: `cd /Users/lsctech/Developer/git-fit && git rm --cached .polaris/graph/graph.sqlite`
Expected output includes: `rm '.polaris/graph/graph.sqlite'`

- [ ] **Step 5: Verify the file still exists on disk and is now ignored**

Run: `ls -la /Users/lsctech/Developer/git-fit/.polaris/graph/graph.sqlite && cd /Users/lsctech/Developer/git-fit && git check-ignore -v .polaris/graph/graph.sqlite`
Expected: `ls` shows the file present; `git check-ignore -v` prints a line like
`.gitignore:<N>:*.sqlite	.polaris/graph/graph.sqlite` (confirms git now ignores it).

- [ ] **Step 6: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add .gitignore
git commit -m "$(cat <<'EOF'
Untrack regenerable graph database from git

.polaris/graph/graph.sqlite is regenerable binary index data.
.polarisignore already excludes *.sqlite for Polaris's own tooling;
git tracking it was the only inconsistency. File stays on disk,
just no longer version-controlled.
EOF
)"
```

Expected: commit succeeds, shows `.gitignore` modified and
`.polaris/graph/graph.sqlite` deleted (from the index only).

---

### Task 2: Remove the stray tracked `=` file

**Files:**
- Remove: `=` (repo root)

**Interfaces:** None — independent of Task 1.

- [ ] **Step 1: Confirm it's tracked and inspect it before deleting**

Run: `cd /Users/lsctech/Developer/git-fit && git ls-files "=" && cat "="`
Expected: `git ls-files` prints `=`; `cat` prints nothing or near-nothing (this is a
classic `command > =` typo artifact — verify it's empty/junk before removing, don't
delete blind).

- [ ] **Step 2: Remove it**

Run: `cd /Users/lsctech/Developer/git-fit && git rm "="`
Expected: `rm '='`

- [ ] **Step 3: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git commit -m "$(cat <<'EOF'
Remove stray '=' artifact file from repo root

Empty file, almost certainly created by a misplaced shell redirect
(e.g. a command with an unquoted > = argument). No content, no
references to it anywhere in the repo.
EOF
)"
```

Expected: commit succeeds, shows `=` deleted.

---

### Task 3: Remove the untracked nested `git-fit/git-fit/` junk directory

**Files:**
- Remove: `git-fit/` (repo root — the untracked nested directory, not the repo itself)

**Interfaces:** None — independent of Tasks 1–2.

**Investigation already performed (2026-07-05), recorded here so the executing agent
doesn't need to redo it:**

```
$ find /Users/lsctech/Developer/git-fit/git-fit -mindepth 1 | wc -l
9
$ find /Users/lsctech/Developer/git-fit/git-fit -type l
(no output — no symlinks)
$ find /Users/lsctech/Developer/git-fit/git-fit -iname ".git" -maxdepth 3
(no output — no nested git repo)
$ du -sh /Users/lsctech/Developer/git-fit/git-fit
8.0K
```

Full contents: `git-fit/.DS_Store`, `git-fit/.github/prompts/` (empty),
`git-fit/AI Training Engine Project/.github/prompts/` (empty), and a recursive
`git-fit/git-fit/git-fit/.github/prompts/` (empty) — i.e. nested copies of the same
empty `.github/prompts/` scaffold, no unique files, no `.git`, no symlinks, 8KB total.
This is inert scaffolding junk (e.g. from a template tool that recursed into its own
output directory), not a checkpoint of unrecovered work.

- [ ] **Step 1: Re-confirm it's still untracked before deleting (state may have
  changed since the investigation above)**

Run: `cd /Users/lsctech/Developer/git-fit && git status --porcelain -- git-fit`
Expected: prints `?? git-fit/` (still untracked, nothing changed)

- [ ] **Step 2: Re-confirm no `.git` and no symlinks (cheap, re-verify before a
  destructive step)**

Run: `find /Users/lsctech/Developer/git-fit/git-fit -iname ".git" -o -type l`
Expected: no output

- [ ] **Step 3: Delete it**

Run: `rm -rf /Users/lsctech/Developer/git-fit/git-fit`
Expected: no output, command exits 0

- [ ] **Step 4: Verify removal and confirm git-fit's own `git status` is unaffected
  (it was untracked, so this should produce no diff)**

Run: `cd /Users/lsctech/Developer/git-fit && ls git-fit 2>&1; git status --porcelain -- git-fit`
Expected: `ls` reports "No such file or directory"; `git status --porcelain` prints
nothing.

No commit needed for this step — the directory was untracked, so there is nothing for
git to record. Note the removal in the Task 4 branch-inventory commit message instead,
or as a standalone note if Task 4 is done separately.

---

### Task 4: Read-only branch inventory

**Files:**
- Create: `smartdocs/runtime/generated/branch-inventory-2026-07-05.md`

**Interfaces:** None — independent of Tasks 1–3. Read-only; produces no changes to any
branch.

- [ ] **Step 1: Generate merged-branch list**

```bash
cd /Users/lsctech/Developer/git-fit
git branch --merged main --format='%(refname:short)' > /tmp/merged.txt
git branch --no-merged main --format='%(refname:short)' > /tmp/unmerged.txt
git branch -r --format='%(refname:short)' | sed 's#^origin/##' > /tmp/remote.txt
```

Expected: three files populated, no errors.

- [ ] **Step 2: Generate last-commit-date and duplicate-group data**

```bash
cd /Users/lsctech/Developer/git-fit
while read -r b; do
  date=$(git log -1 --format=%cd --date=short "$b" 2>/dev/null)
  echo "$b|$date"
done < /tmp/unmerged.txt > /tmp/unmerged-with-dates.txt
sort /tmp/remote.txt | uniq -c | awk '{print $2}' | \
  sed -E 's/-[a-z0-9]{6}$//' | sort | uniq -c | sort -rn | awk '$1>1' > /tmp/duplicate-prefixes.txt
```

Expected: `/tmp/unmerged-with-dates.txt` has one `branch|date` line per unmerged local
branch; `/tmp/duplicate-prefixes.txt` lists any branch-name prefix (with a trailing
6-char hash suffix stripped) that appears more than once among remote branches — this
surfaces the `codex/linear-mention-evos1-283-...` style retry duplicates.

- [ ] **Step 3: Write the report**

Create `smartdocs/runtime/generated/branch-inventory-2026-07-05.md` with:

```markdown
---
kind: raw
status: raw
source: manual audit, 2026-07-05
created: 2026-07-05
source_paths: ""
---

# Branch Inventory — 2026-07-05

Read-only report. No branches were deleted or modified to produce this file.

## Merged into main (safe-delete candidates, not deleted)

<paste contents of /tmp/merged.txt as a bullet list>

## Unmerged (local), with last commit date

<paste contents of /tmp/unmerged-with-dates.txt as a table: Branch | Last commit>

## Remote branch-name prefixes appearing more than once

<paste contents of /tmp/duplicate-prefixes.txt as a table: Count | Prefix — likely
automation retries of the same underlying task>

## Recommendation

This report is for triage only. No branch should be deleted without explicit
per-branch (or per-batch) confirmation from the repo owner, per
`smartdocs/specs/raw/2026-07-05-smartdocs-drift-remediation.md` Section 6, Phase 4.
```

- [ ] **Step 4: Verify the report was written and contains data, not empty sections**

Run: `wc -l /Users/lsctech/Developer/git-fit/smartdocs/runtime/generated/branch-inventory-2026-07-05.md`
Expected: line count > 20 (confirms real content, not a stub)

- [ ] **Step 5: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add smartdocs/runtime/generated/branch-inventory-2026-07-05.md
git commit -m "$(cat <<'EOF'
Add read-only branch inventory report

Lists merged/unmerged/duplicate-prefix branches for owner triage.
No branches deleted or modified — per Phase 4 of the drift
remediation spec, deletion requires explicit per-branch approval.
EOF
)"
```

Expected: commit succeeds, shows one new file.

---

## Verification (whole plan)

- [ ] **Final check: repo is clean and every fix holds**

```bash
cd /Users/lsctech/Developer/git-fit
git status --short                                  # expect clean (or only unrelated pre-existing changes)
git ls-files .polaris/graph/graph.sqlite             # expect no output
git ls-files "="                                     # expect no output
ls git-fit 2>&1                                      # expect "No such file or directory"
git check-ignore -v .polaris/graph/graph.sqlite      # expect a match against *.sqlite
ls smartdocs/runtime/generated/branch-inventory-2026-07-05.md  # expect file present
```

Expected: all five checks pass as annotated.