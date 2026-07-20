---
kind: spec
status: active
source: manual audit, 2026-07-06
created: 2026-07-06
implements: smartdocs/specs/raw/2026-07-05-smartdocs-drift-remediation.md
related: ""
supersedes: ""
superseded_by: ""
depends_on: smartdocs/specs/active/2026-07-05-drift-remediation-phase1-hygiene-plan.md
validates: ""
source_paths: .polaris/skills/,smartdocs/specs/active/,smartdocs/doctrine/,POLARIS_RULES.md,smartdocs/raw/POLARIS_RULES.md
type: spec
---

# Governance Pipeline (Phase 2) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Close the actual gap found in `smartdocs/specs/raw/2026-07-05-smartdocs-drift-remediation.md`
Section 2.B: `docs-review`, `docs-promote`, `docs-triage`, and `docs-ingest` are fully
functional via `npx polaris skill packet <review|promote|triage|ingest>` (the
`@lsctech/polaris` CLI generates a complete authority packet dynamically for each),
but `docs-review`, `docs-promote`, and `docs-triage` have no local `SKILL.md` —
so an agent following `.polaris/skills/ROUTING.md`'s own routing protocol halts with
"Blocking: skill packet not found" before ever reaching the working CLI bootloader.
This plan writes the missing local skill files, reconciles a divergent
`POLARIS_RULES.md` copy, records the authority-model rule the CLI already enforces,
and retires 9 orphaned spec files describing an incompatible pre-CLI ingest design.

**Architecture:** Every new/completed skill file mirrors the exact structure of the
two already-working skills, `polaris-analyze` and `polaris-run`
(`.polaris/skills/polaris-analyze/SKILL.md` and `chain.md`): YAML frontmatter
(`name`/`description`/`role`/`role_file`), a "Command entrypoints" section, a "Polaris
Skill Bootloader" section, and hard-rules sections mirroring the real CLI packet's
`authority_boundaries`/`prohibited_actions`. Step names and chain shapes are drawn from
either an existing `chain.json` (docs-promote) or from real historic run telemetry
(docs-review, docs-triage — both ran successfully on 2026-06-14; the telemetry proves
their actual step names) — nothing is invented without evidence.

**Tech Stack:** Markdown with YAML frontmatter, JSON (`chain.json`), the
`npx polaris` CLI (read-only packet inspection only — this plan does not modify the
`@lsctech/polaris` package itself).

## Global Constraints

- Do not modify the `@lsctech/polaris` npm package or any file under `node_modules/`.
  This plan is entirely repo-local skill/doc files.
- Every new `SKILL.md` must state explicitly that the dynamically-generated packet
  (`npx polaris skill packet <name>`) is the authoritative source and this file is a
  mirror of it as of the date written — per the existing pattern in
  `.polaris/skills/polaris-analyze/SKILL.md` lines 32-34 ("Treat the packet as your
  authoritative instruction source... The packet defines your active role, authority
  boundaries, prohibited actions, deliverables, and stop conditions").
- Do not invent step names, authority boundaries, or prohibited actions not grounded in
  either an existing `chain.json`, real telemetry, or a real `npx polaris skill packet
  <name>` invocation performed as part of this plan.
- `.polaris/roles/` does not exist anywhere in this repo (confirmed via `find`) — even
  `polaris-analyze` and `polaris-run`'s `role_file` fields point at nonexistent files
  (`.polaris/roles/analyst.md`, `.polaris/roles/foreman.md`). This is a pre-existing,
  repo-wide gap. Follow the existing convention (include a `role_file` field) for
  consistency; do not create the missing `.polaris/roles/` directory or its
  contents — that is out of scope for this plan.
- Work happens on the current branch (whatever branch is checked out when this plan is
  executed) — no new branch, no worktree, unless explicitly instructed otherwise at run
  time.

---

### Task 1: `docs-ingest/SKILL.md`

**Files:**
- Create: `.polaris/skills/docs-ingest/SKILL.md`

**Interfaces:** None — independent of all other tasks in this plan. `docs-ingest`
already has a working `chain.json` (5 steps); this task only adds the missing
`SKILL.md` wrapper, it does not touch `chain.json`.

- [ ] **Step 1: Confirm the existing chain.json content (context, not a dependency)**

Run: `cat /Users/lsctech/Developer/git-fit/.polaris/skills/docs-ingest/chain.json`
Expected output (already known, confirming nothing has changed):

```json
{
  "name": "docs-ingest-chain",
  "steps": [
    "01-orient-ingest",
    "02-classify-batch",
    "03-conflict-check",
    "04-place-and-link",
    "05-finalize-ingest"
  ],
  "terminal_steps": ["05-finalize-ingest"],
  "scope": "docs-ingest-only",
  "forbidden_actions": [
    "src file mutations",
    "polaris loop continue",
    "polaris finalize",
    "git push",
    "pr creation",
    "doctrine/active/ writes without user approval",
    "architecture/ writes without user approval",
    "decisions/ writes without user approval",
    "root docs/ writes for new Smart Docs"
  ]
}
```

- [ ] **Step 2: Fetch the live CLI packet to confirm authority_boundaries haven't drifted since 2026-07-06**

Run: `cd /Users/lsctech/Developer/git-fit && npx polaris skill packet ingest`
Expected: a JSON packet with `"skill_name": "ingest"` and `authority_boundaries`
matching (or very close to) the list in Step 3 below. If the packet differs
meaningfully from what's written below, stop and report the discrepancy rather than
writing a `SKILL.md` that contradicts the live packet.

- [ ] **Step 3: Write `.polaris/skills/docs-ingest/SKILL.md`**

```markdown
---
name: docs-ingest
description: Classify and route documents from smartdocs/raw/ into the correct smartdocs/ authority bucket (doctrine/candidate, specs/raw, architecture, decisions, etc.), writing provenance records and updating Polaris map entries. Classification and routing only — does not silently promote to doctrine/active/, specs/active/, architecture/, or decisions/.
role: librarian
role_file: .polaris/roles/librarian.md
---

## Command entrypoints

This skill is the target for the following user commands:

- `docs-ingest`
- `run docs-ingest`

When either of these commands is issued, load this skill packet **first** before any
other action. See `.polaris/skills/ROUTING.md` for the full routing protocol.

---

## Polaris Skill Bootloader

**Before proceeding, you must obtain a skill packet from the Polaris runtime.**

Run the following command:

```
polaris skill packet ingest
```

Note: the CLI's internal skill name is `ingest`, not `docs-ingest` — the routed
command name and the CLI packet name differ. See `.polaris/skills/ROUTING.md`.

- Do not begin work until a packet is returned.
- Treat the packet as your authoritative instruction source.
- The packet defines your active role, authority boundaries, prohibited actions, deliverables, and stop conditions.
- If no packet is produced, stop and report: **Polaris could not authorize this run.**

---

# docs-ingest

Use this skill when the user asks to classify and route new documents out of
`smartdocs/raw/` into the rest of `smartdocs/`.

## How to execute

1. Read `.polaris/skills/docs-ingest/chain.json` — it defines the 5-step chain
   (`01-orient-ingest` … `05-finalize-ingest`) and the terminal step.
2. Read `.taskchain_artifacts/docs-ingest/current-state.json` if present — it contains
   any resumable state from a prior run.
3. Execute steps in the order `chain.json`'s `steps` array defines. Do not skip steps.
4. After every completed step, update `.taskchain_artifacts/docs-ingest/current-state.json`
   before advancing, and append a `step-complete` telemetry event to
   `.taskchain_artifacts/docs-ingest/runs/<run-id>/telemetry.jsonl`.

## Hard rules — what docs-ingest may do

(from the `ingest` packet's `authority_boundaries`, captured 2026-07-06)

- Read documents from `smartdocs/raw/`
- Classify documents by content analysis and front-matter
- Route documents to correct authority directories within `smartdocs/`
- Write provenance records alongside placed files
- Update Polaris map entries to link docs to code areas
- Propose doctrine candidates (route to `doctrine/candidate/` only)
- Emit telemetry events

## Hard rules — what docs-ingest must NOT do

- Write new Smart Docs to root `docs/` — `smartdocs/` is the canonical target
- Silently promote documents to `doctrine/active/`, `specs/active/`, `architecture/`, or `decisions/`
- Mutate source files (`src/`, tests, config)
- Call `polaris loop continue` or `polaris finalize`
- Suppress detected conflicts

**Note:** the packet returned by `polaris skill packet ingest` is generated
dynamically and is always the authoritative, up-to-date source for these rules — this
file mirrors it as of 2026-07-06 for agents that read local skill files first, per
`POLARIS_RULES.md`'s routing protocol. If this file and a freshly fetched packet ever
disagree, the packet wins.
```

- [ ] **Step 4: Verify the file was written correctly**

Run: `cd /Users/lsctech/Developer/git-fit && head -6 .polaris/skills/docs-ingest/SKILL.md`
Expected:
```
---
name: docs-ingest
description: Classify and route documents from smartdocs/raw/ into the correct smartdocs/ authority bucket (doctrine/candidate, specs/raw, architecture, decisions, etc.), writing provenance records and updating Polaris map entries. Classification and routing only — does not silently promote to doctrine/active/, specs/active/, architecture/, or decisions/.
role: librarian
role_file: .polaris/roles/librarian.md
---
```

- [ ] **Step 5: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add .polaris/skills/docs-ingest/SKILL.md
git commit -m "$(cat <<'EOF'
Add missing SKILL.md for docs-ingest

docs-ingest already works via `npx polaris skill packet ingest` and
has a working chain.json, but had no local SKILL.md — so
.polaris/skills/ROUTING.md's own routing protocol would halt with
"Blocking: skill packet not found" before reaching the working CLI
bootloader. Mirrors the polaris-analyze/SKILL.md pattern exactly.
EOF
)"
```

---

### Task 2: `docs-review/chain.json`, `chain.md`, `SKILL.md`

**Files:**
- Create: `.polaris/skills/docs-review/chain.json`
- Create: `.polaris/skills/docs-review/chain.md`
- Create: `.polaris/skills/docs-review/SKILL.md`

**Interfaces:** None — independent of Task 1. `docs-review` currently has no local
files at all (confirmed empty directory, confirmed via `git log --all` that nothing
was ever tracked there).

- [ ] **Step 1: Fetch the live CLI packet**

Run: `cd /Users/lsctech/Developer/git-fit && npx polaris skill packet review`
Expected: a JSON packet with `"skill_name": "review"` and an `authority_boundaries`
list matching Step 4 below. If materially different, stop and report before writing
files that would contradict the live packet.

- [ ] **Step 2: Confirm the proven historic step names from telemetry (context, not a dependency)**

Run: `cat /Users/lsctech/Developer/git-fit/.taskchain_artifacts/docs-review/runs/docs-review-2026-06-14-001/telemetry.jsonl`
Expected (already known):
```
{"event":"run-start","run_id":"docs-review-2026-06-14-001","timestamp":"2026-06-14T22:29:43Z","pending":383}
{"event":"review-complete","run_id":"docs-review-2026-06-14-001","timestamp":"2026-06-14T23:31:22Z","approved":11,"rejected":0,"deferred":372}
{"event":"step-complete","run_id":"docs-review-2026-06-14-001","timestamp":"2026-06-14T23:31:22Z","step":"02-review-packets"}
{"event":"step-complete","run_id":"docs-review-2026-06-14-001","timestamp":"2026-06-14T23:31:22Z","step":"03-apply-decisions","note":"promote required for candidate/ docs"}
{"event":"step-complete","run_id":"docs-review-2026-06-14-001","timestamp":"2026-06-14T23:31:22Z","step":"04-hand-off"}
```
This proves steps `02-review-packets`, `03-apply-decisions`, `04-hand-off` are real.
Step `01` is not directly evidenced in this telemetry (the `run-start` event predates
any `step-complete` event) — name it `01-orient-review` for consistency with
`docs-ingest`'s `01-orient-ingest` and `docs-promote`'s `01-orient-promote` naming
convention, since no telemetry contradicts this and it matches the sibling skills'
pattern exactly.

- [ ] **Step 3: Write `.polaris/skills/docs-review/chain.json`**

```json
{
  "name": "docs-review-chain",
  "steps": [
    "01-orient-review",
    "02-review-packets",
    "03-apply-decisions",
    "04-hand-off"
  ],
  "terminal_steps": ["04-hand-off"],
  "scope": "docs-review-only",
  "forbidden_actions": [
    "src file mutations",
    "polaris loop continue",
    "polaris finalize",
    "git push",
    "pr creation",
    "move, promote, or delete documents directly",
    "auto-approve every packet without reading the document content",
    "suppress or skip flagged packets without recording a decision"
  ]
}
```

- [ ] **Step 4: Write `.polaris/skills/docs-review/chain.md`**

```markdown
---
name: docs-review-chain
description: Route map for docs-review — step order, stop conditions, and artifact requirements for reviewing triaged SmartDocs candidates.
---

# docs-review chain

## Authority

**Polaris runtime state is authoritative. Chat reasoning is not authoritative.**

Query runtime state before acting. Do not infer review scope or prior progress from
conversation context.

## CLI

Always use the repo-local Polaris CLI:

```
polaris <command>
```

Never assume a globally linked `polaris` command exists.

## Step traversal order

```text
01-orient-review     ← load pending review decisions from smartdocs/raw/_triage-queue.json, run-start telemetry
02-review-packets    ← evaluate each flagged packet: read the doc, consider flag type and stale symbols, decide approve/reject/defer
03-apply-decisions   ← write decisions back via `polaris docs review --write-decision <sourcePath> <decision>`; call `polaris docs promote <path>` for approved packets, `polaris doctrine deprecate <path>` for rejected packets
04-hand-off          ← terminal step; summarize decisions reported to user
```

## Stop conditions

**Any step:**
Stop if:
- All packets reviewed (normal completion — proceed to 04).
- An ambiguous packet requires explicit user input.
- An ingest error occurs on apply — report and wait for instruction.

## Run ID format

Format: `docs-review-<date>-<seq>` — matches the proven historic run id
`docs-review-2026-06-14-001`.
- `<date>`: `YYYY-MM-DD`
- `<seq>`: zero-padded sequential number per day (`001`, `002`, …)

## Telemetry

Telemetry file: `.taskchain_artifacts/docs-review/runs/<run-id>/telemetry.jsonl` (append-only).

| Event | Emitted by | Step |
|---|---|---|
| `run-start` | agent | 01 — before reading the triage queue |
| `review-complete` | agent | 03 — after all decisions applied |
| `step-complete` | agent | end of every step |

Required fields on every event: `event`, `run_id`, `timestamp`.

## Artifact authority

`smartdocs/raw/_triage-queue.json` is the sole input queue for pending review
decisions. Decisions are written back into it via
`polaris docs review --write-decision <sourcePath> <decision>` — never edited by hand.

## Never compressed

Always write in full:
- Final decision summary (step 04)
- Any conflict or ambiguity report
```

- [ ] **Step 5: Write `.polaris/skills/docs-review/SKILL.md`**

```markdown
---
name: docs-review
description: Review triaged SmartDocs candidates flagged in smartdocs/raw/_triage-queue.json and decide approve/reject/defer for each, handing approved candidates to polaris docs promote and rejected candidates to polaris doctrine deprecate. Review and decision only — this skill never moves, promotes, or deletes documents directly.
role: librarian
role_file: .polaris/roles/librarian.md
---

## Command entrypoints

This skill is the target for the following user commands:

- `docs-review`
- `run docs-review`

When either of these commands is issued, load this skill packet **first** before any
other action. See `.polaris/skills/ROUTING.md` for the full routing protocol.

---

## Polaris Skill Bootloader

**Before proceeding, you must obtain a skill packet from the Polaris runtime.**

Run the following command:

```
polaris skill packet review
```

Note: the CLI's internal skill name is `review`, not `docs-review` — the routed
command name and the CLI packet name differ. See `.polaris/skills/ROUTING.md`.

- Do not begin work until a packet is returned.
- Treat the packet as your authoritative instruction source.
- The packet defines your active role, authority boundaries, prohibited actions, deliverables, and stop conditions.
- If no packet is produced, stop and report: **Polaris could not authorize this run.**

---

# docs-review

Use this skill when the user asks to review triaged SmartDocs candidates before
promotion.

## How to execute

1. Read `chain.md` — it defines the step order and traversal rules.
2. Read `.taskchain_artifacts/docs-review/current-state.json` if present — it contains
   any resumable state. (No run has left one behind as of 2026-07-06; the most recent
   completed run, `docs-review-2026-06-14-001`, has only a `telemetry.jsonl`, no
   `current-state.json`.)
3. Execute steps in the order `chain.md` defines. Do not skip steps.
4. After every completed step, emit a telemetry event per `chain.md`'s Telemetry table.

## Hard rules — what docs-review may do

(from the `review` packet's `authority_boundaries`, captured 2026-07-06)

- Read `smartdocs/raw/_triage-queue.json` to load pending review decisions
- Read candidate documents from `smartdocs/doctrine/candidate/` to inform decisions
- Evaluate each flagged packet: read the doc, consider the flag type and stale symbols, decide approve/reject/defer
- Write decisions back to the triage queue by calling `polaris docs review --write-decision <sourcePath> <decision>`
- Call `polaris docs promote <path>` for each approved packet (candidate → active)
- Call `polaris doctrine deprecate <path>` for each rejected packet
- Emit telemetry events

## Hard rules — what docs-review must NOT do

- Move, promote, or delete documents directly — decisions must go through `polaris docs promote`/`polaris doctrine deprecate`
- Auto-approve every packet without reading the document content
- Mutate source files (`src/`, tests, config)
- Call `polaris loop continue` or `polaris finalize`
- Suppress or skip flagged packets without recording a decision

**Note:** the packet returned by `polaris skill packet review` is generated
dynamically and is always the authoritative, up-to-date source for these rules — this
file mirrors it as of 2026-07-06 for agents that read local skill files first, per
`POLARIS_RULES.md`'s routing protocol. If this file and a freshly fetched packet ever
disagree, the packet wins.
```

- [ ] **Step 6: Verify all three files are valid**

Run:
```bash
cd /Users/lsctech/Developer/git-fit
python3 -m json.tool .polaris/skills/docs-review/chain.json > /dev/null && echo "chain.json: valid JSON"
head -6 .polaris/skills/docs-review/chain.md
head -6 .polaris/skills/docs-review/SKILL.md
```
Expected: "chain.json: valid JSON" printed; both `head` commands show the expected
YAML frontmatter blocks (`name: docs-review-chain` and `name: docs-review`
respectively).

- [ ] **Step 7: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add .polaris/skills/docs-review/chain.json .polaris/skills/docs-review/chain.md .polaris/skills/docs-review/SKILL.md
git commit -m "$(cat <<'EOF'
Add missing docs-review skill files (chain.json, chain.md, SKILL.md)

docs-review already works via `npx polaris skill packet review` (proven
by the 2026-06-14 telemetry run: 383 pending, 11 approved, 372
deferred) but had zero local files, so ROUTING.md's routing protocol
would halt with "Blocking: skill packet not found" before reaching
the working CLI bootloader. Step names (02-review-packets,
03-apply-decisions, 04-hand-off) are taken directly from that
telemetry; 01-orient-review follows the sibling skills' naming
convention. Mirrors the polaris-analyze pattern.
EOF
)"
```

---

### Task 3: `docs-promote/chain.md`, `SKILL.md`

**Files:**
- Create: `.polaris/skills/docs-promote/chain.md`
- Create: `.polaris/skills/docs-promote/SKILL.md`
- Modify: none (`.polaris/skills/docs-promote/chain.json` already exists and is
  correct — do not change it)

**Interfaces:** None — independent of Tasks 1-2.

- [ ] **Step 1: Confirm the existing chain.json (context, not a dependency)**

Run: `cat /Users/lsctech/Developer/git-fit/.polaris/skills/docs-promote/chain.json`
Expected (already known):
```json
{
  "name": "docs-promote-chain",
  "steps": [
    "01-orient-promote",
    "02-review-candidates",
    "03-read-linked-code",
    "04-conflict-surface",
    "05-await-approval",
    "06-execute-promote-deprecate",
    "07-finalize-promote"
  ],
  "terminal_steps": ["07-finalize-promote"],
  "scope": "docs-promote-only",
  "forbidden_actions": [
    "src file mutations",
    "polaris loop continue",
    "polaris finalize",
    "git push",
    "pr creation",
    "spec-promote --approve without user confirmation in session",
    "doctrine promote without governance field check",
    "architecture/ writes",
    "decisions/ writes"
  ]
}
```

- [ ] **Step 2: Fetch the live CLI packet**

Run: `cd /Users/lsctech/Developer/git-fit && npx polaris skill packet promote`
Expected: `"skill_name": "promote"` with `authority_boundaries` matching Step 4 below.
Stop and report if materially different.

- [ ] **Step 3: Write `.polaris/skills/docs-promote/chain.md`**

```markdown
---
name: docs-promote-chain
description: Route map for docs-promote — step order, stop conditions, and approval-gate enforcement for promoting reviewed SmartDocs candidates to active/deprecated.
---

# docs-promote chain

## Authority

**Polaris runtime state is authoritative. Chat reasoning is not authoritative.**

Query runtime state before acting. Do not infer promotion scope or prior progress from
conversation context.

## CLI

Always use the repo-local Polaris CLI:

```
polaris <command>
```

Never assume a globally linked `polaris` command exists.

## Step traversal order

```text
01-orient-promote            ← run-start telemetry, load current-state.json if resuming
02-review-candidates         ← read smartdocs/raw/ and smartdocs/doctrine/candidate/ to identify promotion candidates
03-read-linked-code          ← read linked source files (from linkedMapArea in the provenance sidecar) to verify relevance
04-conflict-surface          ← read smartdocs/doctrine/active/ and smartdocs/specs/active/ to check for conflicts; call `polaris doctrine spec-promote <path>` WITHOUT --approve to surface the conflict report
05-await-approval            ← STOP; do not proceed without explicit user confirmation of the surfaced conflict report
06-execute-promote-deprecate ← call `polaris doctrine spec-promote <path> --approve` (only after step 05's confirmation) or `polaris doctrine promote <path>` for candidates that pass governance checks; call `polaris doctrine deprecate <path>` for superseded/stale active docs
07-finalize-promote          ← terminal step; report promoted/deprecated/deferred summary
```

## Stop conditions

**Step 05 (await approval):**
Stop unconditionally. Do not advance to step 06 without explicit user confirmation in
the session — this is the approval gate the `promote` CLI packet's own
`prohibited_actions` enforce ("Call --approve without explicit user confirmation in the
session").

**Any step:**
Stop if:
- All candidates reviewed (normal completion).
- An unresolved conflict requires user decision.
- A candidate targets `architecture/` or `decisions/` — those require the explicit ADR
  process, never this chain.

## Run ID format

Format: `docs-promote-<date>-<seq>`.
- `<date>`: `YYYY-MM-DD`
- `<seq>`: zero-padded sequential number per day (`001`, `002`, …)

No historic run exists yet (`.taskchain_artifacts/docs-promote/` has never been
created) — this format follows the sibling skills' convention
(`docs-review-<date>-<seq>`, `docs-ingest-<slug>-<date>-<seq>`).

## Telemetry

Telemetry file: `.taskchain_artifacts/docs-promote/runs/<run-id>/telemetry.jsonl` (append-only).

| Event | Emitted by | Step |
|---|---|---|
| `run-start` | agent | 01 |
| `conflict-report` | agent | 04 |
| `approval-granted` | agent | 05 — only after explicit user confirmation |
| `promote-complete` | agent | 07 |
| `step-complete` | agent | end of every step |

Required fields on every event: `event`, `run_id`, `timestamp`.

## Artifact authority

`.taskchain_artifacts/docs-promote/current-state.json` is the sole authoritative live
state surface once a run exists.

- Update after every completed step before advancing.
- If the update fails: stop and report the persistence failure.

## Never compressed

Always write in full:
- Conflict reports (step 04)
- The approval confirmation record (step 05)
- Final promote/deprecate summary (step 07)
```

- [ ] **Step 4: Write `.polaris/skills/docs-promote/SKILL.md`**

```markdown
---
name: docs-promote
description: Promote reviewed SmartDocs candidates from smartdocs/raw/ or smartdocs/doctrine/candidate/ into doctrine/active or specs/active, or deprecate superseded active docs, after surfacing a conflict report and receiving explicit user approval. Never promotes to architecture/ or decisions/ (those require the explicit ADR process).
role: librarian
role_file: .polaris/roles/librarian.md
---

## Command entrypoints

This skill is the target for the following user commands:

- `docs-promote`
- `run docs-promote`

When either of these commands is issued, load this skill packet **first** before any
other action. See `.polaris/skills/ROUTING.md` for the full routing protocol.

---

## Polaris Skill Bootloader

**Before proceeding, you must obtain a skill packet from the Polaris runtime.**

Run the following command:

```
polaris skill packet promote
```

Note: the CLI's internal skill name is `promote`, not `docs-promote` — the routed
command name and the CLI packet name differ. See `.polaris/skills/ROUTING.md`.

- Do not begin work until a packet is returned.
- Treat the packet as your authoritative instruction source.
- The packet defines your active role, authority boundaries, prohibited actions, deliverables, and stop conditions.
- If no packet is produced, stop and report: **Polaris could not authorize this run.**

---

# docs-promote

Use this skill when the user asks to promote reviewed SmartDocs candidates to active,
or deprecate superseded active docs.

## How to execute

1. Read `chain.md` — it defines the 7-step order, and in particular the mandatory
   approval gate at step 05.
2. Read `.taskchain_artifacts/docs-promote/current-state.json` if present — it
   contains any resumable state.
3. Execute steps in the order `chain.md` defines. Do not skip steps — especially
   step 05, which cannot be skipped or auto-confirmed.
4. After every completed step, update `.taskchain_artifacts/docs-promote/current-state.json`
   before advancing.

## Hard rules — what docs-promote may do

(from the `promote` packet's `authority_boundaries`, captured 2026-07-06)

- Read `smartdocs/raw/` and `smartdocs/doctrine/candidate/` to identify promotion candidates
- Read linked source files (from `linkedMapArea` in the provenance sidecar) to verify relevance
- Read `smartdocs/doctrine/active/` and `smartdocs/specs/active/` to check for conflicts
- Call `polaris doctrine spec-promote <path>` to surface the conflict report (without `--approve`)
- Call `polaris doctrine spec-promote <path> --approve` only after surfacing the report and receiving explicit user confirmation
- Call `polaris doctrine promote <path>` for doctrine candidates that pass governance checks
- Call `polaris doctrine deprecate <path>` for active docs that are superseded or stale
- Emit telemetry events

## Hard rules — what docs-promote must NOT do

- Auto-promote without surfacing the conflict report first
- Call `--approve` without explicit user confirmation in the session
- Mutate source files (`src/`, tests, config)
- Call `polaris loop continue` or `polaris finalize`
- Promote to `architecture/` or `decisions/` — those require the explicit ADR process
- Suppress or ignore detected conflicts

**Note:** the packet returned by `polaris skill packet promote` is generated
dynamically and is always the authoritative, up-to-date source for these rules — this
file mirrors it as of 2026-07-06 for agents that read local skill files first, per
`POLARIS_RULES.md`'s routing protocol. If this file and a freshly fetched packet ever
disagree, the packet wins.
```

- [ ] **Step 5: Verify**

Run:
```bash
cd /Users/lsctech/Developer/git-fit
python3 -m json.tool .polaris/skills/docs-promote/chain.json > /dev/null && echo "chain.json still valid"
head -6 .polaris/skills/docs-promote/chain.md
head -6 .polaris/skills/docs-promote/SKILL.md
```
Expected: "chain.json still valid" printed; both `head` outputs show correct
frontmatter (`name: docs-promote-chain`, `name: docs-promote`).

- [ ] **Step 6: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add .polaris/skills/docs-promote/chain.md .polaris/skills/docs-promote/SKILL.md
git commit -m "$(cat <<'EOF'
Add missing docs-promote chain.md and SKILL.md

docs-promote already works via `npx polaris skill packet promote` and
had a correct 7-step chain.json, but no chain.md or SKILL.md — so
ROUTING.md's routing protocol would halt before reaching the working
CLI bootloader. chain.md documents the existing chain.json's 7 steps
verbatim, including the mandatory step-05 approval gate the promote
packet's own prohibited_actions already enforce. Mirrors the
polaris-analyze pattern.
EOF
)"
```

---

### Task 4: `docs-triage/chain.json`, `chain.md`, `SKILL.md`

**Files:**
- Create: `.polaris/skills/docs-triage/chain.json`
- Create: `.polaris/skills/docs-triage/chain.md`
- Create: `.polaris/skills/docs-triage/SKILL.md`

**Interfaces:** None — independent of Tasks 1-3. `docs-triage` currently has no local
files at all (confirmed empty directory).

- [ ] **Step 1: Fetch the live CLI packet**

Run: `cd /Users/lsctech/Developer/git-fit && npx polaris skill packet triage`
Expected: `"skill_name": "triage"` with `authority_boundaries` matching Step 4 below.
Stop and report if materially different.

- [ ] **Step 2: Confirm the proven historic step names from telemetry (context, not a dependency)**

Run: `cat /Users/lsctech/Developer/git-fit/.taskchain_artifacts/docs-triage/runs/docs-triage-2026-06-14-001/telemetry.jsonl`
Expected (already known):
```
{"event":"run-start","run_id":"docs-triage-2026-06-14-001","timestamp":"2026-06-14T05:14:49Z"}
{"event":"step-complete","run_id":"docs-triage-2026-06-14-001","timestamp":"2026-06-14T05:28:00Z","step":"02-run-triage","flags":383}
{"event":"triage-complete","run_id":"docs-triage-2026-06-14-001","timestamp":"2026-06-14T05:28:00Z","flags_total":383,"flag_types":{"contradiction":0,"duplicate":0,"stale-reference":383}}
{"event":"step-complete","run_id":"docs-triage-2026-06-14-001","timestamp":"2026-06-14T05:28:00Z","step":"03-verify-outputs","outputs":["_triage-queue.json","_triage-report.md"]}
{"event":"step-complete","run_id":"docs-triage-2026-06-14-001","timestamp":"2026-06-14T05:28:00Z","step":"04-hand-off"}
```
This proves steps `02-run-triage`, `03-verify-outputs`, `04-hand-off`. Name the first
step `01-orient-triage` for the same reason given in Task 2 Step 2 (consistent sibling
naming, no contradicting evidence).

- [ ] **Step 3: Write `.polaris/skills/docs-triage/chain.json`**

```json
{
  "name": "docs-triage-chain",
  "steps": [
    "01-orient-triage",
    "02-run-triage",
    "03-verify-outputs",
    "04-hand-off"
  ],
  "terminal_steps": ["04-hand-off"],
  "scope": "docs-triage-only",
  "forbidden_actions": [
    "src file mutations",
    "polaris loop continue",
    "polaris finalize",
    "git push",
    "pr creation",
    "move, promote, or delete any document",
    "auto-approve or auto-reject any triage flag without user review",
    "suppress or ignore detected flags"
  ]
}
```

- [ ] **Step 4: Write `.polaris/skills/docs-triage/chain.md`**

```markdown
---
name: docs-triage-chain
description: Route map for docs-triage — step order, stop conditions, and artifact requirements for flagging doc-vs-doc and doc-vs-code drift ahead of docs-review.
---

# docs-triage chain

## Authority

**Polaris runtime state is authoritative. Chat reasoning is not authoritative.**

Query runtime state before acting. Do not infer triage scope or prior progress from
conversation context.

## CLI

Always use the repo-local Polaris CLI:

```
polaris <command>
```

Never assume a globally linked `polaris` command exists.

## Step traversal order

```text
01-orient-triage     ← run-start telemetry; load smartdocs/doctrine/active/ (canonicals) and smartdocs/doctrine/candidate/ (candidates for triage)
02-run-triage        ← call `polaris docs triage [--dry-run] [--batch-size N] [--resume]` to run the triage pipeline (Phase 1: doc-vs-doc, Phase 2: doc-vs-code)
03-verify-outputs    ← confirm smartdocs/raw/_triage-queue.json and smartdocs/raw/_triage-report.md were written
04-hand-off          ← terminal step; report flag counts by type to user
```

## Stop conditions

**Any step:**
Stop if:
- All documents triaged (normal completion).
- A flag is ambiguous in a way `polaris docs triage` itself cannot resolve — report and
  wait for instruction rather than suppressing it.

## Run ID format

Format: `docs-triage-<date>-<seq>` — matches the proven historic run id
`docs-triage-2026-06-14-001`.
- `<date>`: `YYYY-MM-DD`
- `<seq>`: zero-padded sequential number per day (`001`, `002`, …)

## Telemetry

Telemetry file: `.taskchain_artifacts/docs-triage/runs/<run-id>/telemetry.jsonl` (append-only).

| Event | Emitted by | Step |
|---|---|---|
| `run-start` | agent | 01 |
| `step-complete` | agent | end of every step |
| `triage-complete` | agent | 02 — after the pipeline finishes, with `flags_total` and `flag_types` |

Required fields on every event: `event`, `run_id`, `timestamp`.

## Artifact authority

`smartdocs/raw/_triage-queue.json` (machine-readable flag list, consumed by
`docs-review`) and `smartdocs/raw/_triage-report.md` (human-readable summary) are the
sole outputs of this chain. Per the `triage` packet's own deliverables: "Checkpoint
deleted on successful completion" — do not leave a stale checkpoint behind step 04.

## Never compressed

Always write in full:
- The flag-count summary (step 04)
- Any flag this pipeline could not classify automatically
```

- [ ] **Step 5: Write `.polaris/skills/docs-triage/SKILL.md`**

```markdown
---
name: docs-triage
description: Run the polaris docs triage pipeline to flag doc-vs-doc contradictions/duplicates and doc-vs-code stale references across smartdocs/, writing smartdocs/raw/_triage-queue.json and _triage-report.md for docs-review to consume. Flags only — never moves, promotes, or deletes any document, and never decides approve/reject itself.
role: librarian
role_file: .polaris/roles/librarian.md
---

## Command entrypoints

This skill is the target for the following user commands:

- `docs-triage`
- `run docs-triage`

When either of these commands is issued, load this skill packet **first** before any
other action. See `.polaris/skills/ROUTING.md` for the full routing protocol.

---

## Polaris Skill Bootloader

**Before proceeding, you must obtain a skill packet from the Polaris runtime.**

Run the following command:

```
polaris skill packet triage
```

Note: the CLI's internal skill name is `triage`, not `docs-triage` — the routed
command name and the CLI packet name differ. See `.polaris/skills/ROUTING.md`.

- Do not begin work until a packet is returned.
- Treat the packet as your authoritative instruction source.
- The packet defines your active role, authority boundaries, prohibited actions, deliverables, and stop conditions.
- If no packet is produced, stop and report: **Polaris could not authorize this run.**

---

# docs-triage

Use this skill when the user asks to run doc drift triage ahead of a docs-review pass.

## How to execute

1. Read `chain.md` — it defines the step order and traversal rules.
2. Read `.taskchain_artifacts/docs-triage/current-state.json` if present — it contains
   any resumable state. (No run has left one behind as of 2026-07-06; the most recent
   completed run, `docs-triage-2026-06-14-001`, has only a `telemetry.jsonl`.)
3. Execute steps in the order `chain.md` defines. Do not skip steps.
4. After every completed step, emit a telemetry event per `chain.md`'s Telemetry table.

## Hard rules — what docs-triage may do

(from the `triage` packet's `authority_boundaries`, captured 2026-07-06)

- Read `smartdocs/doctrine/active/` to load canonicals for comparison
- Read `smartdocs/doctrine/candidate/` to load candidates for triage
- Call `polaris docs triage [--dry-run] [--batch-size N] [--resume]` to run the triage pipeline
- Read `smartdocs/raw/_triage-queue.json` and `_triage-report.md` produced by triage
- Emit telemetry events

## Hard rules — what docs-triage must NOT do

- Move, promote, or delete any document — triage flags only, never decides
- Mutate source files (`src/`, tests, config)
- Call `polaris loop continue` or `polaris finalize`
- Suppress or ignore detected flags
- Auto-approve or auto-reject any triage flag without user review

**Note:** the packet returned by `polaris skill packet triage` is generated
dynamically and is always the authoritative, up-to-date source for these rules — this
file mirrors it as of 2026-07-06 for agents that read local skill files first, per
`POLARIS_RULES.md`'s routing protocol. If this file and a freshly fetched packet ever
disagree, the packet wins.
```

- [ ] **Step 6: Verify**

Run:
```bash
cd /Users/lsctech/Developer/git-fit
python3 -m json.tool .polaris/skills/docs-triage/chain.json > /dev/null && echo "chain.json: valid JSON"
head -6 .polaris/skills/docs-triage/chain.md
head -6 .polaris/skills/docs-triage/SKILL.md
```
Expected: "chain.json: valid JSON" printed; both `head` outputs show correct
frontmatter (`name: docs-triage-chain`, `name: docs-triage`).

- [ ] **Step 7: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add .polaris/skills/docs-triage/chain.json .polaris/skills/docs-triage/chain.md .polaris/skills/docs-triage/SKILL.md
git commit -m "$(cat <<'EOF'
Add missing docs-triage skill files (chain.json, chain.md, SKILL.md)

docs-triage already works via `npx polaris skill packet triage`
(proven by the 2026-06-14 telemetry run: 383 flags, all
stale-reference) but had zero local files. Step names
(02-run-triage, 03-verify-outputs, 04-hand-off) are taken directly
from that telemetry; 01-orient-triage follows the sibling skills'
naming convention. Mirrors the polaris-analyze pattern.
EOF
)"
```

---

### Task 5: Document the CLI packet-name mapping in `ROUTING.md`

**Files:**
- Modify: `.polaris/skills/ROUTING.md`

**Interfaces:** None — independent of Tasks 1-4, though it documents a fact those
tasks' `SKILL.md` files each already state individually (this task adds a single
shared quick-reference, not a new source of truth).

- [ ] **Step 1: Confirm the exact insertion point**

Run: `grep -n "^## Required routing protocol" /Users/lsctech/Developer/git-fit/.polaris/skills/ROUTING.md`
Expected: `84:## Required routing protocol` (or similar line number — the section
immediately after the "Recognized command patterns" table and its two `> **Note:**`
callouts).

- [ ] **Step 2: Insert a new subsection right before "## Required routing protocol"**

Using the Edit tool (or equivalent), insert this text immediately before the
`## Required routing protocol` heading, after the existing `> **Note:**` callouts
about `closeout-librarian` and `polaris-medic`:

```markdown

### CLI packet-name mapping

The Polaris CLI's internal skill names (used in `polaris skill packet <name>`) do not
always match the routed command name or the skill directory name. Each skill's own
`SKILL.md` states its exact bootloader command — this table is a quick reference, not
an independent source of truth:

| Routed command | Skill directory | CLI packet name |
|---|---|---|
| `docs-ingest` | `.polaris/skills/docs-ingest/` | `ingest` |
| `docs-review` | `.polaris/skills/docs-review/` | `review` |
| `docs-promote` | `.polaris/skills/docs-promote/` | `promote` |
| `docs-triage` | `.polaris/skills/docs-triage/` | `triage` |
| `polaris-analyze` | `.polaris/skills/polaris-analyze/` | `analyze` |
| `polaris-run` | `.polaris/skills/polaris-run/` | `run` |
```

- [ ] **Step 3: Verify the insertion**

Run: `grep -n "CLI packet-name mapping" /Users/lsctech/Developer/git-fit/.polaris/skills/ROUTING.md`
Expected: one match, at a line number before `## Required routing protocol`.

Run: `grep -c "^|" /Users/lsctech/Developer/git-fit/.polaris/skills/ROUTING.md`
Expected: a count that increased by 8 from before this task (1 header separator + 1
header row + 6 data rows) — confirms the table was actually inserted, not just the
heading.

- [ ] **Step 4: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add .polaris/skills/ROUTING.md
git commit -m "$(cat <<'EOF'
Document CLI packet-name mapping in ROUTING.md

Each of docs-ingest/review/promote/triage's CLI packet name differs
from its routed command name (e.g. docs-review's bootloader is
`polaris skill packet review`, not `docs-review`). Each skill's own
SKILL.md now states this explicitly (added in prior commits this
session), but a shared quick-reference table in the routing doc
itself saves a lookup.
EOF
)"
```

---

### Task 6: Reconcile the two `POLARIS_RULES.md` copies

**Files:**
- Modify: `smartdocs/raw/POLARIS_RULES.md`

**Interfaces:** None — independent of Tasks 1-5.

- [ ] **Step 1: Confirm root is the operative copy**

Run: `grep -n "POLARIS_RULES.md" /Users/lsctech/Developer/git-fit/AGENTS.md /Users/lsctech/Developer/git-fit/CLAUDE.md`
Expected: both files contain `Read [POLARIS_RULES.md](POLARIS_RULES.md) before
beginning any work.` — confirming the repo-root copy, not the `smartdocs/raw/` copy,
is what agents are actually pointed at.

- [ ] **Step 2: Confirm the current diff (already known, re-verify before changing anything)**

Run: `diff -u /Users/lsctech/Developer/git-fit/POLARIS_RULES.md /Users/lsctech/Developer/git-fit/smartdocs/raw/POLARIS_RULES.md`
Expected (already known, from the 2026-07-05 audit): three hunks — (1) a different
one-line file list under "## Repository Overview", (2) the `smartdocs/raw/` copy is
missing the entire "## Graph Navigation" section that exists in the root copy. If the
diff has changed since 2026-07-05, stop and re-assess before overwriting.

- [ ] **Step 3: Overwrite the `smartdocs/raw/` copy with the root copy's content**

Run: `cp /Users/lsctech/Developer/git-fit/POLARIS_RULES.md /Users/lsctech/Developer/git-fit/smartdocs/raw/POLARIS_RULES.md`

- [ ] **Step 4: Verify they're now identical**

Run: `diff /Users/lsctech/Developer/git-fit/POLARIS_RULES.md /Users/lsctech/Developer/git-fit/smartdocs/raw/POLARIS_RULES.md && echo "IDENTICAL"`
Expected: `IDENTICAL` (no diff output before it).

- [ ] **Step 5: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add smartdocs/raw/POLARIS_RULES.md
git commit -m "$(cat <<'EOF'
Reconcile smartdocs/raw/POLARIS_RULES.md with the root copy

The two had diverged (different Repository Overview file list, and
smartdocs/raw/ was missing the entire Graph Navigation section). Root
is the operative copy — both AGENTS.md and CLAUDE.md point agents at
POLARIS_RULES.md, not the smartdocs/raw/ mirror. Overwrote the mirror
to match; this file is bootstrap governance, not doctrine, so there's
no promotion/review step for it — it's kept in sync by direct copy.
EOF
)"
```

---

### Task 7: Record the authority-model rule the `promote` CLI packet already enforces

**Files:**
- Create: `smartdocs/doctrine/active/2026-07-06-docs-promotion-authority-model.md`

**Interfaces:** None — independent of Tasks 1-6, though it documents behavior Task 3
also describes in `docs-promote`'s `chain.md`/`SKILL.md` (this is the durable,
doctrine-level statement; Task 3's files are the operational skill definition).

- [ ] **Step 1: Re-confirm the live packet's approval-gate language (should already be fresh from Task 3 Step 2, but this task can run independently)**

Run: `cd /Users/lsctech/Developer/git-fit && npx polaris skill packet promote`
Expected: `prohibited_actions` includes an entry matching
`"Call --approve without explicit user confirmation in the session"` and one matching
`"Promote to architecture/ or decisions/ — those require explicit ADR process"` (exact
wording may vary slightly by run; capture what's actually returned).

- [ ] **Step 2: Write the doctrine file**

```markdown
---
kind: doctrine
status: active
source: npx polaris skill packet promote (git-fit CLI output, captured 2026-07-06)
doc-type: doctrine
confidence: 0.95
recommended-action: promote
overlap-analysis: No overlap with existing active doctrine — this is the first local record of the docs-promotion approval-gate rule; previously only inferable by invoking the CLI directly.
created: 2026-07-06
implements: ""
related: .polaris/skills/docs-promote/chain.md
supersedes: ""
superseded_by: ""
depends_on: ""
validates: ""
source_paths: .polaris/skills/docs-promote/chain.json,.polaris/skills/docs-promote/chain.md,.polaris/skills/docs-promote/SKILL.md
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
```

- [ ] **Step 3: Verify**

Run: `head -3 /Users/lsctech/Developer/git-fit/smartdocs/doctrine/active/2026-07-06-docs-promotion-authority-model.md`
Expected:
```
---
kind: doctrine
status: active
```

- [ ] **Step 4: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add smartdocs/doctrine/active/2026-07-06-docs-promotion-authority-model.md
git commit -m "$(cat <<'EOF'
Record docs-promotion authority model as local doctrine

Resolves the authority-model gap flagged in the 2026-07-05 drift
spec: rather than citing a different repo's docs-authority-model.md,
this records the actual approval-gate behavior git-fit's own
`npx polaris skill packet promote` already enforces, verified by
invoking it directly on 2026-07-06.
EOF
)"
```

---

### Task 8: Move the 9 orphaned `docs-ingest-step-*` files to `specs/superseded/`

**Files:**
- Move: `smartdocs/specs/active/01-source-intake.md` → `smartdocs/specs/superseded/01-source-intake.md`
- Move: `smartdocs/specs/active/02-classify-lifecycle.md` → `smartdocs/specs/superseded/02-classify-lifecycle.md`
- Move: `smartdocs/specs/active/03-placement-decision.md` → `smartdocs/specs/superseded/03-placement-decision.md`
- Move: `smartdocs/specs/active/04-frontmatter-normalization.md` → `smartdocs/specs/superseded/04-frontmatter-normalization.md`
- Move: `smartdocs/specs/active/05-backlink-normalization.md` → `smartdocs/specs/superseded/05-backlink-normalization.md`
- Move: `smartdocs/specs/active/06-conflict-check.md` → `smartdocs/specs/superseded/06-conflict-check.md`
- Move: `smartdocs/specs/active/07-write-or-move.md` → `smartdocs/specs/superseded/07-write-or-move.md`
- Move: `smartdocs/specs/active/08-index-update.md` → `smartdocs/specs/superseded/08-index-update.md`
- Move: `smartdocs/specs/active/09-final-report.md` → `smartdocs/specs/superseded/09-final-report.md`
- Create: `smartdocs/specs/superseded/README-2026-07-06-docs-ingest-step-files.md`

**Interfaces:** None — independent of Tasks 1-7. Depends only on confirming (Step 1)
that these files are genuinely unreferenced by the current, working `docs-ingest`
(Task 1 already established this — `docs-ingest/chain.json`'s 5 step names share
nothing with these 9 files' names).

- [ ] **Step 1: Re-confirm no current tooling references these 9 files by path (not just by the step-name mismatch already established)**

Run:
```bash
cd /Users/lsctech/Developer/git-fit
grep -rl "specs/active/0[1-9]-\(source-intake\|classify-lifecycle\|placement-decision\|frontmatter-normalization\|backlink-normalization\|conflict-check\|write-or-move\|index-update\|final-report\)" \
  --include="*.json" --include="*.md" \
  --exclude-dir=node_modules --exclude-dir=.git .
```
Expected: no output (no file references these 9 paths directly — they're free-floating,
not linked from any chain.json, SKILL.md, or index). If this returns any matches
outside the 9 files themselves, stop and report before moving anything.

- [ ] **Step 2: Move all 9 files**

```bash
cd /Users/lsctech/Developer/git-fit
git mv smartdocs/specs/active/01-source-intake.md smartdocs/specs/superseded/01-source-intake.md
git mv smartdocs/specs/active/02-classify-lifecycle.md smartdocs/specs/superseded/02-classify-lifecycle.md
git mv smartdocs/specs/active/03-placement-decision.md smartdocs/specs/superseded/03-placement-decision.md
git mv smartdocs/specs/active/04-frontmatter-normalization.md smartdocs/specs/superseded/04-frontmatter-normalization.md
git mv smartdocs/specs/active/05-backlink-normalization.md smartdocs/specs/superseded/05-backlink-normalization.md
git mv smartdocs/specs/active/06-conflict-check.md smartdocs/specs/superseded/06-conflict-check.md
git mv smartdocs/specs/active/07-write-or-move.md smartdocs/specs/superseded/07-write-or-move.md
git mv smartdocs/specs/active/08-index-update.md smartdocs/specs/superseded/08-index-update.md
git mv smartdocs/specs/active/09-final-report.md smartdocs/specs/superseded/09-final-report.md
```

- [ ] **Step 3: Write a README explaining why, for anyone who finds them in `superseded/`**

```markdown
# Why these 9 files were superseded (2026-07-06)

`01-source-intake.md` through `09-final-report.md` describe a 9-step `docs-ingest`
chain (`01-source-intake` … `09-final-report`) built around the pre-smartdocs
`docs/evonotes/` structure, referencing `.evo/routing.md` (which does not exist
anywhere in this repo) and `.codex/skills/docs-ingest/chain.md` (a real path, but not
where this repo's actual `docs-ingest` skill definition lives — that's
`.polaris/skills/docs-ingest/`).

The actual, current `docs-ingest` skill (`.polaris/skills/docs-ingest/chain.json`,
confirmed working via `.taskchain_artifacts/docs-ingest/current-state.json`'s
2026-06-10 completed run) uses a completely different 5-step chain
(`01-orient-ingest`, `02-classify-batch`, `03-conflict-check`, `04-place-and-link`,
`05-finalize-ingest`) that shares no step names with these 9 files.

These files were never connected to the current chain — they were orphaned relics
sitting in `specs/active/` as if current, from a design that predates the smartdocs
migration and the CLI-packet-driven skill model. See
`smartdocs/specs/raw/2026-07-05-smartdocs-drift-remediation.md` Section 2.B for the
full finding.
```

Save as `smartdocs/specs/superseded/README-2026-07-06-docs-ingest-step-files.md`.

- [ ] **Step 4: Verify**

Run:
```bash
cd /Users/lsctech/Developer/git-fit
ls smartdocs/specs/active/ | grep -c "^0[1-9]-"
ls smartdocs/specs/superseded/ | wc -l
```
Expected: first command prints `0` (no more numbered step files in `active/`); second
command prints `10` (9 moved files + the new README).

- [ ] **Step 5: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add smartdocs/specs/superseded/
git commit -m "$(cat <<'EOF'
Move 9 orphaned docs-ingest-step files to specs/superseded/

These described a pre-smartdocs, pre-CLI-packet 9-step docs-ingest
design (referencing docs/evonotes/, a nonexistent .evo/routing.md,
and .codex/skills/) that shares no step names with the actual,
working docs-ingest/chain.json (a different 5-step chain). They were
sitting in specs/active/ as if current. Added a README explaining
why, per the 2026-07-05 drift-remediation spec Section 2.B.
EOF
)"
```

---

### Task 9: Mark the 2026-05-16 EVO doc architecture redesign plan as superseded

**Files:**
- Modify: `smartdocs/doctrine/active/2026-05-16-evo-doc-architecture-redesign.md` (frontmatter only)
- Move: `smartdocs/doctrine/active/2026-05-16-evo-doc-architecture-redesign.md` → `smartdocs/doctrine/deprecated/2026-05-16-evo-doc-architecture-redesign.md`

**Interfaces:** None — independent of Tasks 1-8.

- [ ] **Step 1: Re-confirm the plan's described topology doesn't match reality (already established 2026-07-05, re-verify before changing anything)**

Run: `find /Users/lsctech/Developer/git-fit/docs/EVOnotes -maxdepth 1 -type d`
Expected: subdirectories named `00-index`, `_templates`, `deprecated`, `doctrine`,
`historical`, `implemented`, `needs-review`, `planning-specs` — NOT the plan's
described `spec/{ai,connect,governance,learn,mind,runtime,training}/` topology. This
confirms the plan describes a structure that was never built as written.

- [ ] **Step 2: Read the current frontmatter to know exactly what to change**

Run: `head -10 /Users/lsctech/Developer/git-fit/smartdocs/doctrine/active/2026-05-16-evo-doc-architecture-redesign.md`
Expected:
```
---
doc-type: architecture
confidence: 0.90
recommended-action: promote
overlap-analysis: Reviewed by docs-review-agent; approved via triage queue
---
```

- [ ] **Step 3: Update the frontmatter**

Using the Edit tool, replace:

```yaml
---
doc-type: architecture
confidence: 0.90
recommended-action: promote
overlap-analysis: Reviewed by docs-review-agent; approved via triage queue
---
```

with:

```yaml
---
kind: doctrine
status: deprecated
doc-type: architecture
confidence: 0.90
recommended-action: deprecate
overlap-analysis: Reviewed by docs-review-agent; approved via triage queue
superseded_by: ""
deprecation-reason: >
  Describes a docs/evonotes/spec/{ai,connect,governance,learn,mind,runtime,training}/
  topology that was never built as written. Current docs/EVOnotes/ (confirmed
  2026-07-06) uses a completely different, much smaller structure
  (00-index/implemented/needs-review/planning-specs/historical/doctrine/deprecated).
  smartdocs/ superseded this plan's entire premise sometime between 2026-05-16 and
  smartdocs' earliest ingest run (2026-06-04). No single doc supersedes this one by
  name; it is superseded in practice by smartdocs/ as a whole. See
  smartdocs/specs/raw/2026-07-05-smartdocs-drift-remediation.md Section 1.
---
```

- [ ] **Step 4: Move the file to `deprecated/`**

```bash
cd /Users/lsctech/Developer/git-fit
git mv smartdocs/doctrine/active/2026-05-16-evo-doc-architecture-redesign.md smartdocs/doctrine/deprecated/2026-05-16-evo-doc-architecture-redesign.md
```

- [ ] **Step 5: Verify**

Run: `head -12 /Users/lsctech/Developer/git-fit/smartdocs/doctrine/deprecated/2026-05-16-evo-doc-architecture-redesign.md`
Expected: frontmatter shows `status: deprecated` and the `deprecation-reason` field
from Step 3.

- [ ] **Step 6: Commit**

```bash
cd /Users/lsctech/Developer/git-fit
git add smartdocs/doctrine/deprecated/2026-05-16-evo-doc-architecture-redesign.md
git commit -m "$(cat <<'EOF'
Mark 2026-05-16 EVO doc architecture redesign plan as deprecated

Describes a docs/evonotes/spec/{ai,connect,...} topology that was
never built as written — current docs/EVOnotes/ uses a completely
different structure, and smartdocs/ superseded this plan's entire
premise in practice sometime between 2026-05-16 and smartdocs' first
ingest run (2026-06-04). It was still sitting in doctrine/active/ and
still cited by docs/POLARIS.md as current structural authority.
EOF
)"
```

**Note for the executing agent:** after this task, `docs/POLARIS.md` still cites this
file's old path (`smartdocs/doctrine/active/2026-05-16-evo-doc-architecture-redesign.md`)
as current structural authority. Updating `docs/POLARIS.md` itself is Phase 3 scope
(legacy `docs/` migration), not this plan — do not fix it here.

---

## Verification (whole plan)

- [ ] **Final check: all four skills load a packet, all local files are internally consistent**

```bash
cd /Users/lsctech/Developer/git-fit
for skill in ingest review promote triage; do
  echo "=== $skill ==="
  npx polaris skill packet "$skill" > /dev/null && echo "packet OK" || echo "packet FAILED"
done
python3 -m json.tool .polaris/skills/docs-review/chain.json > /dev/null && echo "docs-review chain.json valid"
python3 -m json.tool .polaris/skills/docs-triage/chain.json > /dev/null && echo "docs-triage chain.json valid"
python3 -m json.tool .polaris/skills/docs-promote/chain.json > /dev/null && echo "docs-promote chain.json valid (unchanged)"
python3 -m json.tool .polaris/skills/docs-ingest/chain.json > /dev/null && echo "docs-ingest chain.json valid (unchanged)"
diff POLARIS_RULES.md smartdocs/raw/POLARIS_RULES.md && echo "POLARIS_RULES.md copies identical"
ls smartdocs/specs/active/ | grep -c "^0[1-9]-"    # expect 0
ls smartdocs/doctrine/deprecated/2026-05-16-evo-doc-architecture-redesign.md  # expect: file exists
```

Expected: all "packet OK", all "valid" messages, "copies identical", `0`, and the
deprecated file path printed with no error.
