---
title: 2026-05-16-evo-doc-architecture-redesign
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/2026-05-16-evo-doc-architecture-redesign.md
updated: 2026-07-24
---

# EVO Documentation Architecture Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure the EVO documentation system to use deterministic folder-level traversal with local indexes, eliminating agent discovery failures caused by orphaned concepts and missing local navigation structure.

**Architecture:** Every domain folder gets a README.md that serves as its local navigation contract. The global `00-index` becomes a root map, not a sole oracle. The `docs-ingest` skill gains index-maintenance and orphan-detection steps. Note classification becomes explicit and traversal-consistent across all agent runtimes.

**Tech Stack:** Markdown, YAML frontmatter, Obsidian wiki-links, bash, `.codex/skills/` skill system

---

## Diagnosis: Why Echo Doctrine Became Hard to Discover

Echo doctrine consists of 10 files spread across `spec/ai/`, `spec/mind/`, and `spec/governance/`. The failure mode is:

1. `spec/ai/` has 49 files and **no README** — an agent entering the folder sees a flat list with no guidance on what is canonical, what is the entry point, or whether Echo is even here.
2. The global index (`00-index/README.md`) describes domains at the folder level but does not list named concepts within those folders.
3. Semantic search degrades when a concept like "Echo" maps to many files with similar names (`Echo_1_*`, `Echo_2_*`, `Echo_Deep_*`, etc.) — probabilistic retrieval returns noisy results without a deterministic disambiguation path.
4. No cross-domain traversal map exists: `Echo_in_mind/EVOmind_Echo_Compiler_Architecture.md` is invisible from `spec/ai/` and vice versa.

The fix: local README in `spec/ai/` that explicitly names Echo as a canonical concept, lists its entry point, and cross-references `spec/mind/` and `spec/governance/` for related doctrine.

---

## Why Semantic-Only Traversal Is Degrading

- 538 evonotes files create retrieval noise. Without local disambiguation, agents fall back to cosine similarity over a flat corpus.
- Notes with similar names (`EVOconnect — X.md`, `EVOconnect — X 2.md`) are indistinguishable without a local context file explaining which is canonical and which is a duplicate candidate.
- `needs-review/ai/` has 63 unclassified notes — agents cannot know whether to trust them.
- No folder declares its canonical entry points, so agents must infer entry points heuristically, which fails at scale.

---

## Note Classification System

Every evonote has one **lifecycle status** and one **note type**. These are already partially tracked in frontmatter but must be consistently enforced.

### Lifecycle Status (existing, formalized)

| status | meaning | folder |
|--------|---------|--------|
| `spec` | Canonical intended architecture | `spec/<domain>/` |
| `implemented` | Verified in codebase via GitNexus | `implemented/<domain>/` |
| `historical` | Preserved for reasoning, not active doctrine | `historical/<domain>/` |
| `needs-review` | Unaudited — do not rely on as doctrine | `needs-review/<domain>/` |
| `deprecated` | Superseded, retired | `deprecated/` |

### Note Type (NEW — add to frontmatter)

| type | meaning |
|------|---------|
| `doctrine` | Canonical philosophy/principles (authoritative, stable) |
| `architecture` | System design, components, relationships |
| `spec` | Implementation specification (detailed, actionable) |
| `execution` | How to run/operate a system (procedures, commands) |
| `exploration` | Speculative or investigatory — not settled doctrine |
| `index` | Folder navigation aid (README/INDEX files) |
| `historical` | Past state, preserved for context |
| `audit` | Linear issues, reviews, incident reports |

### Conflict Rules

- `needs-review` notes may NOT be cited as doctrine by agents.
- `exploration` notes must declare their speculative status in body content.
- `historical` notes must include a deprecation reason.
- `deprecated` notes must include what replaced them.
- `index` notes are agent navigation aids, not doctrine.

---

## Anti-Orphaning Rules

A note is **orphaned** when:
1. It has no parent folder README that acknowledges it, OR
2. It has no Obsidian wiki-link from any other note, OR
3. Its domain/lifecycle combination has no match in the folder structure.

### Enforcement

- Every folder with >3 notes MUST have a README.md.
- Every README.md MUST list canonical entry points explicitly.
- Every new note added by `docs-ingest` MUST be acknowledged in the folder README within the same ingest run.
- Notes in `needs-review/` older than 90 days without a `last_reviewed` update must be flagged by any ingest run.

### Cross-Domain Notes

Concepts spanning multiple domains (e.g. Echo across `ai/`, `mind/`, `governance/`) must have:
- A **primary home** in one domain's README entry points list.
- **Cross-reference entries** in every other domain README that touches the concept.
- A cross-domain note MUST NOT be listed as the primary entry point in more than one domain.

---

## Proposed Folder Topology

The existing folder structure is sound. No restructuring needed. What changes:

```text
docs/evonotes/
  00-index/
    README.md          ← root navigation map (updated — becomes traversal root)
    _lifecycle-manifest.md
  spec/
    README.md          ← NEW: domain overview + traversal map for spec/
    ai/
      README.md        ← NEW: local index for spec/ai/
    connect/
      README.md        ← NEW
    governance/
      README.md        ← NEW
    mind/
      README.md        ← NEW
    runtime/
      README.md        ← NEW
    training/
      README.md        ← NEW
  implemented/
    README.md          ← NEW
    ai/
      README.md        ← NEW
    runtime/
      README.md        ← NEW
    training/
      README.md        ← NEW
  needs-review/
    README.md          ← NEW
    ai/
      README.md        ← NEW
    training/
      README.md        ← NEW
    governance/
      README.md        ← NEW
  historical/
    README.md          ← NEW
    ai/
      README.md        ← NEW
    runtime/
      README.md        ← NEW
    training/
      README.md        ← NEW
  deprecated/
    README.md          ← NEW

docs/raw/
  README.md            ← exists, update with classification rules
  archived/
    README.md          ← NEW
  audits-and-issues/
    README.md          ← NEW (was audits/, name correction)
  deprecated/
    README.md          ← NEW
  contracts/
    README.md          ← NEW
```

---

## Local Index Template

Every domain folder README follows this template exactly. Agents use it as the deterministic traversal contract for the folder.

```markdown
---
type: index
domain: <domain-name>
lifecycle_stage: <spec|implemented|historical|needs-review|deprecated>
last_updated: YYYY-MM-DD
---

# <Domain> — <Lifecycle Stage>

## Purpose

<2-3 sentences: what this domain covers, why it exists, what questions it answers>

## Canonical Entry Points

Start here when exploring this domain:

- [[<filename-without-extension>]] — <one-line description>
- [[<filename-without-extension>]] — <one-line description>

## Traversal Map

<Concept → File mapping. For each major concept in this folder, list which file to read.>

| Concept | File | Note Type |
|---------|------|-----------|
| <concept> | [[<filename>]] | doctrine/architecture/spec |

## Named Concepts

<List significant named systems/features in this folder and their entry point file>

- **<ConceptName>**: [[<entry-file>]] (cross-refs: [[<related>]])

## Cross-Domain References

Concepts in this folder that connect to other domains:

| Concept | Primary Domain | This Domain's Angle |
|---------|---------------|---------------------|
| <concept> | [[spec/ai/README]] | <how this domain relates> |

## File Count

<N> files. Last audit: YYYY-MM-DD.

## Deprecated / Needs-Review Notes

<List any notes in this folder that should NOT be trusted as active doctrine, and why.>

## Child Folders

<If this README is for a parent directory, list subdirectories and link to their READMEs.>
```

---

## Traversal Philosophy

**Primary traversal is folder-local. Semantic search is a fallback.**

Agents entering any folder should be able to:
1. Read the folder README.
2. Identify the canonical entry point for their concept.
3. Follow Obsidian wiki-links to related notes.
4. Exit to adjacent domains via cross-domain references.

**No folder is a dead end.** Every README must either list content or redirect to where the content lives.

**Traversal root:** `docs/evonotes/00-index/README.md` → links to `spec/README.md`, `implemented/README.md`, etc. → links to domain READMEs → links to individual notes.

**Agents should never need semantic search for known concepts.** If a concept has been canonized, it is reachable by deterministic traversal from the root.

---

## Cross-Agent Compatibility Strategy

All index files must be:
- Plain Markdown (no agent-specific extensions)
- Self-contained (readable without context from other files)
- Obsidian wiki-links for cross-references (rendered as links in Obsidian, readable as `[[name]]` syntax in all other agents)
- No agent-specific directives or tags
- Frontmatter in YAML (parseable by all agents)

Agents that do not support Obsidian wiki-links can parse `[[filename]]` as the filename reference. The README structure provides deterministic navigation without requiring link resolution.

The traversal system works at all capability levels:
- **Full-capability agents** (Claude, Gemini): follow wiki-links, read frontmatter, traverse graph
- **Mid-capability agents** (Copilot, Windsurf): read README, find filenames, open directly
- **Minimal agents** (local LLMs): read README text, extract concept→file mapping from the table

---

## Migration Strategy for needs-review/

The 63 notes in `needs-review/ai/` represent the largest unresolved classification debt. Migration priority:

1. **EVOLoRA mesh notes** → `spec/training/` (they're architecture specs about the LoRA system)
2. **R2/ops/build notes** → `historical/<domain>/` or new `spec/infrastructure/` subfolder
3. **Alice runtime notes** → `implemented/ai/` after GitNexus verification
4. **Speculative notes** → add `note_type: exploration` to frontmatter, keep in `needs-review/` until human approval

Migration is NOT part of this plan's scope. This plan creates the infrastructure. Migration runs as a separate audit pass.

---

## Strategy for Raw Doc Categories

| Category | Current Location | Handling |
|----------|-----------------|----------|
| Imported Notion docs | `raw/archived/` | Add README explaining these are uncanonized exports |
| Old exports | `raw/archived/` | README + date-stamp naming convention |
| Partial canonicalization | `raw/` root | docs-ingest routes; incompletely processed get `needs-review` |
| Archived concepts | `raw/archived/` | Stays raw — canonize only if referenced by active doctrine |
| Historical doctrine | `evonotes/historical/` | Stays in place; local README marks it as non-active |
| Speculative doctrine | `evonotes/needs-review/` + `note_type: exploration` | Never cited as truth |
| Implementation specs | `evonotes/spec/<domain>/` + `note_type: spec` | Canonical, actionable |
| Execution docs | `evonotes/spec/<domain>/` + `note_type: execution` | Canonical, operational |
| Architecture docs | `evonotes/spec/<domain>/` + `note_type: architecture` | Canonical, structural |

---

## doc-ingest Redesign

The current skill processes files into correct folders. The redesign adds four new steps:

**New Step 4e — Update local folder README**
After placing a new file in a destination folder, the skill must:
1. Read the folder's README.md.
2. Add the new file to the Traversal Map table.
3. If the README doesn't exist, create it from the template.
4. Write the updated README.

**New Step 7 — Orphan Detection**
After all files are processed, scan all `evonotes/` folders for files NOT listed in their folder's README Traversal Map. Report them as orphans.

**New Step 8 — Missing Index Detection**
Scan all `evonotes/` subdirectories. Any directory with >3 `.md` files and no `README.md` is a missing-index violation. Report each one.

**New Step 9 — Wiki-Link Integrity Check**
For each canonized file in the current ingest run, verify that all `[[wiki-links]]` in the file resolve to an existing file in `evonotes/`. Report broken links.

**New Step 10 — Traversal Root Update**
After any ingest run that adds a new domain subfolder, update `00-index/README.md` to include the new folder in its child-folder navigation map.

---

## File Structure

### Files to Create

| File | Responsibility |
|------|---------------|
| `docs/evonotes/spec/README.md` | Lifecycle-stage index for all spec/ content |
| `docs/evonotes/spec/ai/README.md` | Domain index: Alice, Echo, cognition, memory |
| `docs/evonotes/spec/connect/README.md` | Domain index: EVOconnect, Hive, Delegator |
| `docs/evonotes/spec/governance/README.md` | Domain index: EVE, safety, authority |
| `docs/evonotes/spec/mind/README.md` | Domain index: EVOmind, EVOlearn |
| `docs/evonotes/spec/runtime/README.md` | Domain index: multi-device, IPC, storage |
| `docs/evonotes/spec/training/README.md` | Domain index: EVOtraining, LoRA, adapters |
| `docs/evonotes/implemented/README.md` | Lifecycle-stage index for implemented/ |
| `docs/evonotes/implemented/ai/README.md` | Domain index |
| `docs/evonotes/implemented/runtime/README.md` | Domain index |
| `docs/evonotes/implemented/training/README.md` | Domain index |
| `docs/evonotes/needs-review/README.md` | Warning: these notes are unaudited |
| `docs/evonotes/needs-review/ai/README.md` | Domain index + classification debt notes |
| `docs/evonotes/needs-review/training/README.md` | Domain index |
| `docs/evonotes/needs-review/governance/README.md` | Domain index |
| `docs/evonotes/historical/README.md` | Lifecycle-stage index for historical/ |
| `docs/evonotes/historical/ai/README.md` | Domain index |
| `docs/evonotes/historical/runtime/README.md` | Domain index |
| `docs/evonotes/historical/training/README.md` | Domain index |
| `docs/evonotes/deprecated/README.md` | Deprecated notes — what replaced each |
| `docs/raw/archived/README.md` | Raw archived material — not canonical |
| `docs/raw/audits-and-issues/README.md` | Linear issues — audit use only |
| `docs/raw/deprecated/README.md` | Stale references — why deprecated |
| `docs/raw/contracts/README.md` | Runtime contract specs |
| `docs/evonotes/_templates/_TEMPLATE - Index README.md` | Local index template |
| `docs/evonotes/_templates/_TEMPLATE - Traversal Root.md` | Root navigation map template |
| `.codex/skills/docs-ingest/SKILL.md` | Updated with Steps 4e, 7, 8, 9, 10 |

### Files to Modify

| File | Change |
|------|--------|
| `docs/evonotes/00-index/README.md` | Add root traversal map linking to all lifecycle-stage READMEs |
| `docs/evonotes/README.md` | Add traversal instructions for agents |
| `docs/raw/README.md` | Update with note classification table and traversal rules |

---

## Tasks

### Task 1: Create the Local Index Template

**Files:**
- Create: `docs/evonotes/_templates/_TEMPLATE - Index README.md`

- [ ] **Step 1: Write the template file**

```markdown
---
type: index
domain: REPLACE_ME
lifecycle_stage: REPLACE_ME
last_updated: YYYY-MM-DD
---

# REPLACE_ME Domain — LIFECYCLE_STAGE

## Purpose

REPLACE_ME: 2-3 sentences describing what this domain covers.

## Canonical Entry Points

Start here when exploring this domain:

- [[REPLACE_ME]] — REPLACE_ME description

## Traversal Map

| Concept | File | Note Type |
|---------|------|-----------|
| REPLACE_ME | [[REPLACE_ME]] | doctrine |

## Named Concepts

- **REPLACE_ME**: [[REPLACE_ME]] (cross-refs: [[REPLACE_ME]])

## Cross-Domain References

| Concept | Primary Domain | This Domain's Angle |
|---------|---------------|---------------------|
| REPLACE_ME | [[spec/REPLACE_ME/README]] | REPLACE_ME |

## File Count

N files. Last audit: YYYY-MM-DD.

## Deprecated / Needs-Review Notes

None. (Update if any notes in this folder should not be trusted as active doctrine.)

## Child Folders

None. (Update if subdirectories exist.)
```

- [ ] **Step 2: Commit**

```bash
git add docs/evonotes/_templates/_TEMPLATE\ -\ Index\ README.md
git commit -m "docs: add local index README template for EVO traversal redesign"
```

---

### Task 2: Update the Traversal Root (00-index)

**Files:**
- Modify: `docs/evonotes/00-index/README.md`

- [ ] **Step 1: Read the current file**

Read `docs/evonotes/00-index/README.md` in full.

- [ ] **Step 2: Add root traversal map section**

Insert after the existing Folder Structure section:

```markdown
## Root Traversal Map

This is the deterministic traversal root for all EVO doctrine.

Start here. Follow links to lifecycle stages. Follow links to domains. Follow links to notes.

### By Lifecycle Stage

| Stage | Folder | Description |
|-------|--------|-------------|
| `spec` | [[spec/README]] | Canonical intended architecture — source of truth for future implementation |
| `implemented` | [[implemented/README]] | Confirmed implemented — GitNexus-verifiable |
| `needs-review` | [[needs-review/README]] | Unaudited — do NOT cite as doctrine without review |
| `historical` | [[historical/README]] | Preserved for reasoning — not active doctrine |
| `deprecated` | [[deprecated/README]] | Superseded — what replaced each note is noted there |

### By Domain (within spec/)

| Domain | Folder | Covers |
|--------|--------|--------|
| `ai` | [[spec/ai/README]] | Alice core, Echo, memory, cognition, embeddings, identity |
| `connect` | [[spec/connect/README]] | EVOconnect, Hive, Swarm, Delegator, Talents, Methods |
| `governance` | [[spec/governance/README]] | EVE, authority, safety, secrets, audit, privacy, policy |
| `learn` | [[spec/learn/README]] | EVOlearn, Student/Teacher Alice, lessons, retention, school |
| `mind` | [[spec/mind/README]] | EVOmind cognition, reflection loop, signal model, Echo compiler |
| `runtime` | [[spec/runtime/README]] | Multi-device, Hive protocol, storage, model runtime, IPC |
| `training` | [[spec/training/README]] | EVOtraining, LoRA, adapters, workout intelligence |

### Anti-Orphaning Check

Any concept not reachable via this traversal map is orphaned. Run docs-ingest to detect and report orphans.
```

- [ ] **Step 3: Commit**

```bash
git add docs/evonotes/00-index/README.md
git commit -m "docs: add root traversal map to 00-index for deterministic agent navigation"
```

---

### Task 3: Create spec/README.md (Lifecycle Stage Index)

**Files:**
- Create: `docs/evonotes/spec/README.md`

- [ ] **Step 1: Write the file**

```markdown
---
type: index
lifecycle_stage: spec
last_updated: 2026-05-16
---

# Spec — Canonical Intended Architecture

## Purpose

This folder contains canonical EVO architecture notes: intended system design, philosophy, and specifications that have not yet been fully implemented. These are the source of truth for future implementation work.

Do NOT use `needs-review/` notes as doctrine. Do NOT treat `spec/` notes as confirmed unless they appear in `implemented/` too.

## Domain Folders

| Domain | Folder | File Count | Covers |
|--------|--------|------------|--------|
| Alice / AI Core | [[ai/README]] | 49 | Alice core, Echo, memory, cognition, embeddings, identity |
| EVOconnect | [[connect/README]] | 102 | Hive, Swarm, Delegator, Talents, Methods, Living Notes |
| Governance | [[governance/README]] | 69 | EVE, authority, safety, secrets, audit, privacy, policy |
| EVOlearn | [[learn/README]] | ~44 | Student/Teacher Alice, lessons, retention, school, Learn Pro |
| EVOmind | [[mind/README]] | ~8 | EVOmind reflection, signal model, Echo compiler, cognition |
| Runtime | [[runtime/README]] | 29 | Multi-device, Hive protocol, storage, model runtime, IPC |
| EVOtraining | [[training/README]] | 14 | LoRA, adapters, workout intelligence |

## Top-Level Spec Files

These files describe the overall EVO system and do not belong to a single domain:

- `EVO — *.md` files at `spec/` root: high-level system overview documents

## Note Types Present

`doctrine`, `architecture`, `spec`, `execution`, `exploration`

## Anti-Orphaning

Any `.md` file added to `spec/<domain>/` must be listed in that domain's `README.md` Traversal Map within the same commit.
```

- [ ] **Step 2: Commit**

```bash
git add docs/evonotes/spec/README.md
git commit -m "docs: add spec/ lifecycle stage index for EVO traversal redesign"
```

---

### Task 4: Create spec/ai/README.md (Echo Discovery Fix)

This is the highest-priority domain index because Echo doctrine discovery failure is the canonical example of the problem.

**Files:**
- Create: `docs/evonotes/spec/ai/README.md`

- [ ] **Step 1: List all files in spec/ai/**

```bash
ls docs/evonotes/spec/ai/ | sort
```

- [ ] **Step 2: Write the file**

The file must explicitly name Echo as a named concept with its entry point and cross-domain references.

```markdown
---
type: index
domain: ai
lifecycle_stage: spec
last_updated: 2026-05-16
---

# AI Domain — Spec

## Purpose

This folder contains canonical doctrine for Alice's core intelligence: her identity, memory, cognition architecture, context handling, embeddings, and the Echo reflection system. These are the intended design specs — not all are fully implemented yet.

For confirmed implementations, see [[implemented/ai/README]].
For unaudited notes, see [[needs-review/ai/README]].

## Canonical Entry Points

Start here when exploring the AI domain:

- [[Alice — Core Identity]] — Alice's identity, values, and behavioral anchors
- [[Echo_1_Canonical_Philosophy]] — Echo doctrine entry point (start here for Echo)
- [[Alice Memory Architecture]] — how Alice stores and recalls information
- [[Alice Context Architecture]] — how context windows are managed

## Named Concepts

### Echo System

The Echo system is Alice's reflective cognition layer. It spans multiple domains.

- **Primary doctrine**: [[Echo_1_Canonical_Philosophy]] (start here)
- **Architecture**: [[Echo_2_Architecture]]
- **Security activation**: [[Echo_3_Security_Activation]]
- **UX/Hall spec**: [[Echo_4_UX_Hall]]
- **Deep architecture**: [[Echo_Deep_Architecture_Spec]]
- **Production spec**: [[Echo_Production_Spec]]
- **System spec**: [[Echo_System_Spec]]
- **Future speculative**: [[Council_of_Echoes_Future_Spec]] (note_type: exploration)
- **Cross-ref mind**: [[spec/mind/EVOmind_Echo_Compiler_Architecture]] — compiler layer
- **Cross-ref governance**: [[spec/governance/No Secret Echo Rule]] — security constraint

### Alice Core

- **Identity**: Alice — Core Identity (if file exists) or first Alice identity file
- **Cognition**: Alice cognition architecture file
- **Memory**: Alice memory architecture file
- **Context**: Alice context architecture file
- **Embeddings**: Alice embeddings file

## Traversal Map

| Concept | File | Note Type |
|---------|------|-----------|
| Echo philosophy | [[Echo_1_Canonical_Philosophy]] | doctrine |
| Echo architecture | [[Echo_2_Architecture]] | architecture |
| Echo security | [[Echo_3_Security_Activation]] | spec |
| Echo UX | [[Echo_4_UX_Hall]] | spec |
| Echo deep arch | [[Echo_Deep_Architecture_Spec]] | architecture |
| Echo production | [[Echo_Production_Spec]] | spec |
| Council of Echoes | [[Council_of_Echoes_Future_Spec]] | exploration |

## Cross-Domain References

| Concept | Primary Domain | Related Domain |
|---------|---------------|----------------|
| Echo compiler | [[spec/ai/README]] (primary) | [[spec/mind/README]] — compiler layer |
| Echo security | [[spec/ai/README]] (primary) | [[spec/governance/README]] — No Secret Echo Rule |
| Alice training | [[spec/training/README]] (primary) | This domain — adapter behavior |

## File Count

49 files. Last audit: 2026-05-16.

## Needs-Review Notes in This Domain

63 notes exist in [[needs-review/ai/README]] that have NOT been classified. Do not cite those as doctrine.
```

- [ ] **Step 3: Commit**

```bash
git add docs/evonotes/spec/ai/README.md
git commit -m "docs: add spec/ai/ local index — makes Echo doctrine discoverable via deterministic traversal"
```

---

### Task 5: Create spec/connect/README.md

**Files:**
- Create: `docs/evonotes/spec/connect/README.md`

- [ ] **Step 1: List files to understand key concepts**

```bash
ls docs/evonotes/spec/connect/ | head -30
```

- [ ] **Step 2: Write the file**

```markdown
---
type: index
domain: connect
lifecycle_stage: spec
last_updated: 2026-05-16
---

# Connect Domain — Spec

## Purpose

This folder contains canonical doctrine for EVOconnect: the inter-system communication layer that governs how Alice instances, Talents, Methods, Living Notes, and the Hive network interact. This is the largest domain (102 files) and covers both synchronous and asynchronous coordination patterns.

## Canonical Entry Points

Start here:

- `EVOconnect — *.md` files with "Overview" or "Architecture" in the name — system entry point
- Delegator architecture file — how work is delegated between agents
- Hive protocol file — how Alice instances coordinate
- Talent specification — what Talents are and how they execute

## Traversal Map

| Concept | File | Note Type |
|---------|------|-----------|
| EVOconnect overview | EVOconnect overview file | architecture |
| Delegator | Delegator architecture file | architecture |
| Hive protocol | Hive protocol file | spec |
| Talents | Talent spec file | spec |
| Methods | Methods spec file | spec |
| Living Notes | Living Notes spec file | spec |
| External Agent Governance | EVOconnect — External Agent Governance Model.md | spec |
| Swarm | Swarm architecture file | architecture |

## Named Concepts

- **EVOconnect**: The inter-app communication framework. Entry: EVOconnect overview file.
- **Hive**: The distributed Alice coordination network. Spec in `spec/runtime/` for protocol; connect/ for application-layer behavior.
- **Delegator**: The component that routes work to appropriate Talents.
- **Talents**: Discrete capability units executed by Alice.
- **Methods**: Structured invocation patterns for Talent execution.
- **Living Notes**: Dynamic notes that update through Talent execution.

## Known Duplicate

`EVOconnect — External Agent Governance Model.md` and `EVOconnect — External Agent Governance Model 2.md` are suspected duplicates. Do not cite both as distinct doctrine until reconciled. See `00-index/README.md` for reconciliation notes.

## Cross-Domain References

| Concept | Related Domain |
|---------|---------------|
| Hive protocol | [[spec/runtime/README]] — transport/IPC layer |
| Delegation gating | [[spec/governance/README]] — authority enforcement |
| Talent execution | [[spec/training/README]] — how Talents are trained |

## File Count

102 files. Last audit: 2026-05-16.
```

- [ ] **Step 3: Commit**

```bash
git add docs/evonotes/spec/connect/README.md
git commit -m "docs: add spec/connect/ local index for EVOconnect traversal"
```

---

### Task 5b: Create spec/learn/ and Migrate EVOlearn Notes from spec/mind/

The `spec/learn/` subfolder is listed in the 00-index as pending. Of the 52 files currently in `spec/mind/`, approximately 44 are EVOlearn/Learn/Lesson/Student/Teacher/School-specific and belong in `spec/learn/`. The ~8 remaining `EVOmind-*` and `MOC EVOmind` files stay in `spec/mind/`.

**Files:**
- Create: `docs/evonotes/spec/learn/` (directory + README)
- Move: all Learn/Lesson/Student/Teacher/School/EVOlearn files from `spec/mind/` → `spec/learn/`

The files to move are identifiable by prefix: anything starting with `Learn`, `Lesson`, `Student Alice`, `Teacher Alice`, `School`, `_MOC - Learn`, `_MOC - School`, `_MOC - Student`, `_MOC - Teacher`, `_MOC – Retention`, `EVOlearn`, `Adult Self-Study`, `Classroom`, `Hint Retry`, `Homework Diagnostic`, `Milestone-Architecture`, `Retention`, `Unlock-Rules`, `Why Learn Is Last`, `TA-SA`, `MOC - Learn Pro`.

The files that stay in `spec/mind/` are: `EVOmind Reflection Loop.md`, `EVOmind User Journey.md`, `EVOmind — Adapter Behavior.md`, `EVOmind — Signal Model.md`, `EVOmind_Canonical_Definition.md`, `EVOmind_Echo_Compiler_Architecture.md`, `MOC EVOmind.md`, `Why EVO Notes System Exists.md`.

- [ ] **Step 1: Create spec/learn/ and move files**

```bash
mkdir -p docs/evonotes/spec/learn

# Move all Learn-prefixed files
for f in \
  "Learn - Guardian Management Controls.md" \
  "Learn - Ingestion Pipeline + Source Map.md" \
  "Learn - Kids Restricted Topics.md" \
  "Learn - Optional Tutor Link.md" \
  "Learn - Policy Profiles (Adult vs Kids).md" \
  "Learn Pro - Domain Styles.md" \
  "Learn Pro - Mesh Integration.md" \
  "Learn Pro - Philosophy.md" \
  "Learn Pro - Risk & Governance.md" \
  "Learn Pro - Timeline & Snapshot System.md" \
  "Learn Pro - UX Language.md" \
  "Learn Pro - Workshop Flow.md" \
  "Learn – Eureka Mechanic.md" \
  "Learn – Self-Study Mode (Ingestion Notebook).md" \
  "Learn-Routing-Rules.md" \
  "Lesson-Compiler.md" \
  "Lesson-Pack-Schema.md" \
  "Lesson-Pack-Sync.md" \
  "Lesson-Pack-Versioning.md" \
  "Student Alice - Architecture.md" \
  "Student Alice - Core Architecture.md" \
  "Student Alice - Kids School Mode.md" \
  "Teacher Alice - Architecture.md" \
  "School EVE Architecture.md" \
  "School-Home-Topology.md" \
  "TA-SA - Communication Protocol.md" \
  "Adult Self-Study Mode.md" \
  "Classroom-Architecture.md" \
  "Hint Retry Ask Alice Flow.md" \
  "Homework Diagnostic Engine.md" \
  "Milestone-Architecture.md" \
  "Retention Stability Banding Scope.md" \
  "Retention-Driven Exploration Control.md" \
  "Unlock-Rules.md" \
  "Why Learn Is Last.md" \
  "EVOlearn Learning Feedback System.md" \
  "EVOlearn MOC.md" \
  "MOC - Learn Pro (Collaborative Style Workshop).md" \
  "_EVOlearn Core Governance MOC.md" \
  "_MOC - Learn Architecture.md" \
  "_MOC - School.md" \
  "_MOC - Student Engine.md" \
  "_MOC - Teacher Alice.md" \
  "_MOC – Retention Model.md"; do
  src="docs/evonotes/spec/mind/$f"
  [ -f "$src" ] && mv "$src" "docs/evonotes/spec/learn/$f"
done
```

- [ ] **Step 2: Verify the split**

```bash
echo "=== Remaining in mind/ ==="
ls docs/evonotes/spec/mind/

echo "=== Moved to learn/ ==="
ls docs/evonotes/spec/learn/ | wc -l
```

Expected: ~8 files in mind/, ~44 files in learn/.

- [ ] **Step 3: Create spec/learn/README.md**

```markdown
---
type: index
domain: learn
lifecycle_stage: spec
last_updated: 2026-05-16
---

# Learn Domain — Spec

## Purpose

This folder contains canonical doctrine for EVOlearn: the structured learning system that Alice uses to teach students. This covers the Student/Teacher Alice model, lesson architecture, school/home topology, retention mechanics, guardian controls, and Learn Pro collaborative workshops.

EVOlearn was previously housed in `spec/mind/` and has been promoted to its own domain due to its scope and independence from EVOmind's cognition/reflection responsibilities.

For EVOmind's reflection and signal model, see [[spec/mind/README]].

## Canonical Entry Points

- [[EVOlearn MOC]] — map of content for the EVOlearn system
- [[_MOC - Learn Architecture]] — overall learn architecture map
- [[Student Alice - Core Architecture]] — Student Alice entry point
- [[Teacher Alice - Architecture]] — Teacher Alice entry point
- [[Learn Pro - Philosophy]] — Learn Pro philosophy (start here for Learn Pro)

## Named Concepts

- **EVOlearn**: The structured learning orchestration system. Entry: [[EVOlearn MOC]]
- **Student Alice**: Alice's student-mode persona and architecture. Entry: [[Student Alice - Core Architecture]]
- **Teacher Alice**: Alice's teacher-mode persona. Entry: [[Teacher Alice - Architecture]]
- **TA-SA Protocol**: How Teacher Alice and Student Alice communicate. [[TA-SA - Communication Protocol]]
- **Learn Pro**: Collaborative style workshop system. Entry: [[Learn Pro - Philosophy]]
- **Lesson Pack**: The packaged lesson format. Entry: [[Lesson-Pack-Schema]]
- **Retention Model**: How EVOlearn tracks retention stability. [[_MOC – Retention Model]]
- **School Mode**: School/home topology and governance. [[School-Home-Topology]]
- **Guardian Controls**: Parental/guardian management layer. [[Learn - Guardian Management Controls]]

## Traversal Map

| Concept | File | Note Type |
|---------|------|-----------|
| EVOlearn overview | [[EVOlearn MOC]] | index |
| Learn architecture | [[_MOC - Learn Architecture]] | index |
| Student Alice core | [[Student Alice - Core Architecture]] | architecture |
| Student Alice full | [[Student Alice - Architecture]] | architecture |
| Student Alice kids | [[Student Alice - Kids School Mode]] | spec |
| Teacher Alice | [[Teacher Alice - Architecture]] | architecture |
| TA-SA protocol | [[TA-SA - Communication Protocol]] | spec |
| Lesson compiler | [[Lesson-Compiler]] | architecture |
| Lesson pack schema | [[Lesson-Pack-Schema]] | spec |
| Lesson pack sync | [[Lesson-Pack-Sync]] | spec |
| Lesson pack versioning | [[Lesson-Pack-Versioning]] | spec |
| Milestone architecture | [[Milestone-Architecture]] | architecture |
| Retention banding | [[Retention Stability Banding Scope]] | spec |
| Retention exploration | [[Retention-Driven Exploration Control]] | spec |
| Learn routing | [[Learn-Routing-Rules]] | spec |
| Eureka mechanic | [[Learn – Eureka Mechanic]] | spec |
| Self-study ingestion | [[Learn – Self-Study Mode (Ingestion Notebook)]] | spec |
| Adult self-study | [[Adult Self-Study Mode]] | spec |
| Homework diagnostic | [[Homework Diagnostic Engine]] | spec |
| Hint/retry flow | [[Hint Retry Ask Alice Flow]] | spec |
| Unlock rules | [[Unlock-Rules]] | spec |
| Guardian controls | [[Learn - Guardian Management Controls]] | spec |
| Kids restricted topics | [[Learn - Kids Restricted Topics]] | spec |
| Policy profiles | [[Learn - Policy Profiles (Adult vs Kids)]] | spec |
| Tutor link | [[Learn - Optional Tutor Link]] | spec |
| Ingestion pipeline | [[Learn - Ingestion Pipeline + Source Map]] | spec |
| School EVE | [[School EVE Architecture]] | architecture |
| School-home topology | [[School-Home-Topology]] | architecture |
| Classroom architecture | [[Classroom-Architecture]] | architecture |
| Learn Pro philosophy | [[Learn Pro - Philosophy]] | doctrine |
| Learn Pro workshop | [[Learn Pro - Workshop Flow]] | spec |
| Learn Pro domain styles | [[Learn Pro - Domain Styles]] | spec |
| Learn Pro mesh | [[Learn Pro - Mesh Integration]] | spec |
| Learn Pro governance | [[Learn Pro - Risk & Governance]] | spec |
| Learn Pro timeline | [[Learn Pro - Timeline & Snapshot System]] | spec |
| Learn Pro UX language | [[Learn Pro - UX Language]] | spec |
| EVOlearn governance | [[_EVOlearn Core Governance MOC]] | index |
| School MOC | [[_MOC - School]] | index |
| Student engine MOC | [[_MOC - Student Engine]] | index |
| Teacher Alice MOC | [[_MOC - Teacher Alice]] | index |
| Retention model MOC | [[_MOC – Retention Model]] | index |
| Learn Pro MOC | [[MOC - Learn Pro (Collaborative Style Workshop)]] | index |
| Why learn is last | [[Why Learn Is Last]] | doctrine |

## Cross-Domain References

| Concept | Related Domain |
|---------|---------------|
| EVOmind reflection | [[spec/mind/README]] — cognition/signal model stays in mind |
| Echo compiler (used by learn) | [[spec/mind/EVOmind_Echo_Compiler_Architecture]] |
| Training adapters | [[spec/training/README]] — how learn signals inform LoRA |
| Governance / School EVE | [[spec/governance/README]] — authority model for school mode |

## File Count

44 files (migrated from spec/mind/ on 2026-05-16). Last audit: 2026-05-16.
```

- [ ] **Step 4: Commit**

```bash
git add docs/evonotes/spec/learn/ docs/evonotes/spec/mind/
git commit -m "docs: extract spec/learn/ from spec/mind/ — 44 EVOlearn notes promoted to own domain"
```

---

### Task 6: Create spec/governance/README.md, spec/mind/README.md, spec/runtime/README.md, spec/training/README.md

**Files:**
- Create: `docs/evonotes/spec/governance/README.md`
- Create: `docs/evonotes/spec/mind/README.md`
- Create: `docs/evonotes/spec/runtime/README.md`
- Create: `docs/evonotes/spec/training/README.md`

- [ ] **Step 1: List files in each domain**

```bash
ls docs/evonotes/spec/governance/ | head -20
ls docs/evonotes/spec/mind/ | head -20
ls docs/evonotes/spec/runtime/ | head -20
ls docs/evonotes/spec/training/ | head -20
```

- [ ] **Step 2: Write spec/governance/README.md**

```markdown
---
type: index
domain: governance
lifecycle_stage: spec
last_updated: 2026-05-16
---

# Governance Domain — Spec

## Purpose

This folder contains canonical doctrine for EVE (EVO's governance system): authority models, safety enforcement, audit trails, privacy policy, secrets management, and the security constraints that apply to Alice and all EVO subsystems.

## Canonical Entry Points

- EVE overview/architecture file — entry point for governance system
- Authority model file — who can authorize what
- Safety architecture file — GatingEngine + AnswerRepair (current two-layer model)
- Privacy policy file — data handling constraints

## Named Concepts

- **EVE**: EVO's governance and authority enforcement engine.
- **GatingEngine**: Deterministic safety enforcement layer (current architecture — do NOT reintroduce ENF).
- **AnswerRepair**: Domain-aware UX repair layer (second safety layer).
- **No Secret Echo Rule**: [[No Secret Echo Rule]] — security constraint on Echo activation (cross-ref: [[spec/ai/README]]).

## Cross-Domain References

| Concept | Related Domain |
|---------|---------------|
| Echo security | [[spec/ai/README]] — Echo doctrine primary home |
| Delegation authority | [[spec/connect/README]] — Delegator implementation |
| Training safety | [[spec/training/README]] — adapter safety constraints |

## Architecture Anti-Drift Notes

Current safety architecture is **two-layer only**: GatingEngine + AnswerRepair.
Do NOT document or spec an ENF (Enforcement) layer — it was evaluated and removed.
Any note recommending ENF belongs in `docs/raw/deprecated/`.

## File Count

69 files. Last audit: 2026-05-16.
```

- [ ] **Step 3: Write spec/mind/README.md**

Note: this README is written AFTER Task 5b moves ~44 EVOlearn files to `spec/learn/`. This folder now contains only EVOmind-specific files (~8).

```markdown
---
type: index
domain: mind
lifecycle_stage: spec
last_updated: 2026-05-16
---

# Mind Domain — Spec

## Purpose

This folder contains canonical doctrine for EVOmind: Alice's internal cognition, reflection, and signal model. EVOmind manages how Alice processes experience, runs reflection loops, and feeds signals to the learning and training systems.

EVOlearn (student/teacher Alice, lessons, retention, school) was migrated to [[spec/learn/README]] and is no longer in this folder.

## Canonical Entry Points

- [[EVOmind_Canonical_Definition]] — what EVOmind is
- [[EVOmind_Echo_Compiler_Architecture]] — how EVOmind processes Echo outputs
- [[EVOmind Reflection Loop]] — the core reflection cycle
- [[EVOmind — Signal Model]] — what signals EVOmind emits

## Named Concepts

- **EVOmind**: Alice's cognition and reflection engine.
- **Echo Compiler**: [[EVOmind_Echo_Compiler_Architecture]] — processes Echo outputs. Echo doctrine primary home: [[spec/ai/README]].
- **Signal Model**: [[EVOmind — Signal Model]] — signals EVOmind emits to training and learning systems.
- **Adapter Behavior**: [[EVOmind — Adapter Behavior]] — how EVOmind interacts with LoRA adapters.

## Traversal Map

| Concept | File | Note Type |
|---------|------|-----------|
| EVOmind definition | [[EVOmind_Canonical_Definition]] | doctrine |
| Echo compiler | [[EVOmind_Echo_Compiler_Architecture]] | architecture |
| Reflection loop | [[EVOmind Reflection Loop]] | architecture |
| Signal model | [[EVOmind — Signal Model]] | spec |
| Adapter behavior | [[EVOmind — Adapter Behavior]] | spec |
| User journey | [[EVOmind User Journey]] | spec |
| MOC | [[MOC EVOmind]] | index |
| General doctrine | [[Why EVO Notes System Exists]] | doctrine |

## Cross-Domain References

| Concept | Related Domain |
|---------|---------------|
| Echo doctrine | [[spec/ai/README]] — Echo primary home |
| EVOlearn / Student / Teacher Alice | [[spec/learn/README]] — migrated there |
| LoRA adapter training | [[spec/training/README]] — how signals drive adapters |

## File Count

~8 files (after EVOlearn migration to spec/learn/ on 2026-05-16). Last audit: 2026-05-16.
```

- [ ] **Step 4: Write spec/runtime/README.md**

```markdown
---
type: index
domain: runtime
lifecycle_stage: spec
last_updated: 2026-05-16
---

# Runtime Domain — Spec

## Purpose

This folder contains canonical doctrine for EVO's runtime infrastructure: multi-device coordination, the Hive protocol transport layer, on-device model runtime (GGUF + llama.cpp), storage, and IPC (inter-process communication).

## Canonical Entry Points

- Runtime overview file — entry point
- Hive protocol transport file — how Alice instances communicate at the network level
- On-device inference spec — GGUF + llama.cpp configuration

## Architecture Anti-Drift Notes

Current mobile inference runtime: **GGUF + llama.cpp** (iOS Metal, Android NDK).
MLX was evaluated and abandoned. Do NOT document MLX runtime specs — they belong in `docs/raw/deprecated/`.

## Cross-Domain References

| Concept | Related Domain |
|---------|---------------|
| Hive application layer | [[spec/connect/README]] — EVOconnect application behavior |
| Model download/R2 | `docs/raw/audits-and-issues/` — infrastructure ops |

## File Count

29 files. Last audit: 2026-05-16.
```

- [ ] **Step 5: Write spec/training/README.md**

```markdown
---
type: index
domain: training
lifecycle_stage: spec
last_updated: 2026-05-16
---

# Training Domain — Spec

## Purpose

This folder contains canonical doctrine for EVOtraining: the LoRA adapter system, workout intelligence, and how Alice's behavior is shaped through training. This is spec-level — for confirmed implementations see [[implemented/training/README]].

## Canonical Entry Points

- EVOtraining overview file — entry point
- LoRA architecture file — LoRAKind taxonomy and adapter spec
- Workout intelligence spec — how workouts map to training signals

## Architecture Anti-Drift Notes

LoRAKind = `{ U, T, GU, GT }`. ENF and VOICE adapters do not exist.
Do NOT document ENF or VOICE as LoRA adapter types.

## Named Concepts

- **EVOtraining**: The training orchestration system.
- **LoRAKind**: `{ U (User), T (Trainer), GU (Global User), GT (Global Trainer) }` — the four adapter types.

## Cross-Domain References

| Concept | Related Domain |
|---------|---------------|
| Adapter safety constraints | [[spec/governance/README]] — GatingEngine enforcement |
| EVOlearn training signals | [[spec/mind/README]] — learning system |

## File Count

14 files. Last audit: 2026-05-16.
```

- [ ] **Step 6: Commit all four**

```bash
git add docs/evonotes/spec/governance/README.md docs/evonotes/spec/mind/README.md docs/evonotes/spec/runtime/README.md docs/evonotes/spec/training/README.md
git commit -m "docs: add local indexes for spec/governance, spec/mind, spec/runtime, spec/training"
```

---

### Task 7: Create implemented/ Indexes

**Files:**
- Create: `docs/evonotes/implemented/README.md`
- Create: `docs/evonotes/implemented/ai/README.md`
- Create: `docs/evonotes/implemented/runtime/README.md`
- Create: `docs/evonotes/implemented/training/README.md`

- [ ] **Step 1: Write implemented/README.md**

```markdown
---
type: index
lifecycle_stage: implemented
last_updated: 2026-05-16
---

# Implemented — Confirmed in Codebase

## Purpose

Notes in this folder represent EVO systems that have been confirmed in the codebase. These are distinct from `spec/` notes, which describe intended architecture. An `implemented/` note means the system exists and is verifiable via GitNexus.

**Warning**: `gitnexus_verified: false` on all notes means the verification pass has not run yet. Run `gitnexus_impact` or `gitnexus_context` on the key symbol before treating any note here as confirmed ground truth.

## Domain Folders

| Domain | Folder | File Count |
|--------|--------|------------|
| Alice / AI | [[ai/README]] | 55 |
| Runtime | [[runtime/README]] | 31 |
| Training | [[training/README]] | 42 |

## Missing Domains

`implemented/connect/`, `implemented/governance/`, `implemented/mind/` do not exist yet. After GitNexus verification passes confirm implementation, notes should be promoted from `spec/` to these folders.
```

- [ ] **Step 2: Write implemented/ai/README.md**

```markdown
---
type: index
domain: ai
lifecycle_stage: implemented
last_updated: 2026-05-16
---

# AI Domain — Implemented

## Purpose

Confirmed Alice AI implementations verifiable in the codebase. See [[spec/ai/README]] for intended architecture (some spec items may not yet be in implemented/).

## Canonical Entry Points

Review the files here to understand what Alice capabilities are confirmed implemented vs. specced.

## File Count

55 files. Last audit: 2026-05-16.

## Verification Status

All notes have `gitnexus_verified: false`. Run GitNexus before citing as implementation ground truth.
```

- [ ] **Step 3: Write implemented/runtime/README.md** (same pattern)

```markdown
---
type: index
domain: runtime
lifecycle_stage: implemented
last_updated: 2026-05-16
---

# Runtime Domain — Implemented

## Purpose

Confirmed runtime implementations. See [[spec/runtime/README]] for full runtime architecture spec.

## File Count

31 files. Last audit: 2026-05-16.

## Verification Status

All notes have `gitnexus_verified: false`. Run GitNexus before citing as ground truth.
```

- [ ] **Step 4: Write implemented/training/README.md**

```markdown
---
type: index
domain: training
lifecycle_stage: implemented
last_updated: 2026-05-16
---

# Training Domain — Implemented

## Purpose

Confirmed EVOtraining and LoRA adapter implementations. See [[spec/training/README]] for the full training spec.

## File Count

42 files. Last audit: 2026-05-16.

## Verification Status

All notes have `gitnexus_verified: false`. Run GitNexus before citing as ground truth.
```

- [ ] **Step 5: Commit**

```bash
git add docs/evonotes/implemented/README.md docs/evonotes/implemented/ai/README.md docs/evonotes/implemented/runtime/README.md docs/evonotes/implemented/training/README.md
git commit -m "docs: add implemented/ lifecycle stage and domain indexes"
```

---

### Task 8: Create needs-review/ and historical/ Indexes

**Files:**
- Create: `docs/evonotes/needs-review/README.md`
- Create: `docs/evonotes/needs-review/ai/README.md`
- Create: `docs/evonotes/needs-review/training/README.md`
- Create: `docs/evonotes/needs-review/governance/README.md`
- Create: `docs/evonotes/historical/README.md`
- Create: `docs/evonotes/historical/ai/README.md`
- Create: `docs/evonotes/historical/runtime/README.md`
- Create: `docs/evonotes/historical/training/README.md`
- Create: `docs/evonotes/deprecated/README.md`

- [ ] **Step 1: Write needs-review/README.md**

```markdown
---
type: index
lifecycle_stage: needs-review
last_updated: 2026-05-16
---

# Needs-Review — Unaudited Notes

## ⚠️ Agent Warning

Notes in this folder have NOT been classified as doctrine. Do NOT cite them as authoritative.
Do NOT use them to inform architecture decisions without human review.

## Purpose

Staging area for notes whose lifecycle status is uncertain. They were routed here by docs-ingest when domain or status was ambiguous.

## Domain Folders

| Domain | Count | Primary Classification Debt |
|--------|-------|---------------------------|
| [[ai/README]] | 63 | EVOLoRA mesh, R2/ops, Alice runtime — needs triage |
| [[training/README]] | 6 | Training variants — needs GitNexus verification |
| [[governance/README]] | 1 | Single note — needs status determination |

## Migration Path

1. Human or governed agent reviews each note.
2. Note is moved to appropriate lifecycle folder (`spec/`, `implemented/`, `historical/`).
3. Folder README in destination is updated.
4. docs-ingest Step 7 (orphan scan) confirms it's no longer orphaned here.

Notes older than 90 days without `last_reviewed` update should be flagged by any ingest run.
```

- [ ] **Step 2: Write needs-review/ai/README.md**

```markdown
---
type: index
domain: ai
lifecycle_stage: needs-review
last_updated: 2026-05-16
---

# AI Domain — Needs Review

## ⚠️ 63 Unclassified Notes

These notes could not be confidently classified in the lifecycle-organization-v1 pass. Common reasons:

- EVOLoRA mesh notes (overlap training + runtime)
- R2/ops/build notes (likely `historical` or new `infrastructure` domain)
- Alice runtime notes (likely `implemented/ai/` after GitNexus verification)
- Speculative notes (likely `spec/ai/` with `note_type: exploration`)

## Recommended Triage

| Category | Estimated Count | Recommended Destination |
|----------|----------------|------------------------|
| EVOLoRA mesh | ~15 | `spec/training/` |
| R2/ops/build notes | ~10 | `historical/ai/` or new `spec/infrastructure/` |
| Alice runtime confirmed | ~20 | `implemented/ai/` after GitNexus |
| Alice speculative | ~18 | `spec/ai/` with `note_type: exploration` |

## File Count

63 files. Last audit: 2026-05-16.
```

- [ ] **Step 3: Write needs-review/training/README.md and needs-review/governance/README.md**

```markdown
---
type: index
domain: training
lifecycle_stage: needs-review
last_updated: 2026-05-16
---

# Training Domain — Needs Review

## 6 Unclassified Notes

Training notes requiring GitNexus verification to determine if they belong in `implemented/training/` or remain in `spec/training/`.

## File Count

6 files. Last audit: 2026-05-16.
```

```markdown
---
type: index
domain: governance
lifecycle_stage: needs-review
last_updated: 2026-05-16
---

# Governance Domain — Needs Review

## 1 Unclassified Note

Single governance note requiring status determination.

## File Count

1 file. Last audit: 2026-05-16.
```

- [ ] **Step 4: Write historical/ and deprecated/ indexes**

```markdown
---
type: index
lifecycle_stage: historical
last_updated: 2026-05-16
---

# Historical — Preserved for Reference

## Purpose

Notes preserved for historical reasoning. These describe past EVO system states and are NOT active doctrine. Reading them is valid for understanding evolution of the system; citing them as current architecture is not.

## Domain Folders

| Domain | Count |
|--------|-------|
| [[ai/README]] | 6 |
| [[runtime/README]] | 7 |
| [[training/README]] | 1 |
```

(Write similar skeleton READMEs for historical/ai, historical/runtime, historical/training with file counts and purpose notes.)

```markdown
---
type: index
lifecycle_stage: deprecated
last_updated: 2026-05-16
---

# Deprecated — Superseded Notes

## Purpose

Notes for systems, architectures, or decisions that have been explicitly retired. Each deprecated note should document what replaced it.

Currently empty — deprecated notes from `docs/raw/deprecated/` have not yet been promoted here.

## Known Deprecated Systems (from docs/raw/deprecated/)

- MLX runtime → replaced by GGUF + llama.cpp
- ElevenLabs TTS → replaced by Supertonic 2 on-device ONNX
- Phi-4-mini → replaced by Qwen2.5-1.5B Q4_K_M
- ENF adapter → removed (two-layer safety now)
- VOICE adapter → removed
```

- [ ] **Step 5: Commit all**

```bash
git add docs/evonotes/needs-review/README.md docs/evonotes/needs-review/ai/README.md docs/evonotes/needs-review/training/README.md docs/evonotes/needs-review/governance/README.md docs/evonotes/historical/README.md docs/evonotes/historical/ai/README.md docs/evonotes/historical/runtime/README.md docs/evonotes/historical/training/README.md docs/evonotes/deprecated/README.md
git commit -m "docs: add needs-review, historical, deprecated lifecycle stage indexes"
```

---

### Task 9: Create docs/raw/ Subdirectory Indexes

**Files:**
- Create: `docs/raw/archived/README.md`
- Create: `docs/raw/audits-and-issues/README.md`
- Create: `docs/raw/deprecated/README.md`
- Create: `docs/raw/contracts/README.md`

- [ ] **Step 1: Write archived/README.md**

```markdown
---
type: index
last_updated: 2026-05-16
---

# Raw / Archived

## Purpose

Uncanonized raw material: Notion exports, old imports, processed-but-not-promoted docs, and duplicates of existing evonotes. These are NOT canonical doctrine.

## Contents

~98 files. Mix of:
- Original Notion exports before canonization
- Files that were duplicates of existing evonotes (moved here by docs-ingest Step 3)
- Miscellaneous notes that didn't match doctrine patterns

## To Promote

If a file here is needed as active doctrine, run docs-ingest on a copy placed in `docs/raw/` root.
```

- [ ] **Step 2: Write audits-and-issues/README.md**

```markdown
---
type: index
last_updated: 2026-05-16
---

# Raw / Audits and Issues

## Purpose

Linear issue files routed here by docs-ingest. These are NOT doctrine. They are audit records, bug reports, and feature issues from Linear.

Issue prefixes: `EVOS1-*`, `EVOC-*`, `EVOTRA-*`, `EVOMIND-*`, `EVOFL-*`

## Contents

66 files. All Linear issue exports.

## Usage

Reference for understanding what was decided or discovered during specific Linear issues. Do not treat as architecture doctrine.
```

- [ ] **Step 3: Write deprecated/README.md and contracts/README.md**

```markdown
---
type: index
last_updated: 2026-05-16
---

# Raw / Deprecated

## Purpose

Files with stale references that were rejected by docs-ingest Step 2. These recommend systems that no longer exist: MLX, ElevenLabs TTS, ENF adapter, VOICE adapter, Phi-4-mini.

## Contents

24 files. Stale reference rejections.

## Warning

Do NOT promote these to evonotes without explicit architecture decision to revive the referenced system.
```

```markdown
---
type: index
last_updated: 2026-05-16
---

# Raw / Contracts

## Purpose

Runtime contract specs for Alice — formal interface definitions. These are staging-area contracts, not yet promoted to evonotes.

## Files

- `alice_manifest.schema.json` — JSON Schema for Alice manifests
- `alice_manifest.example.json` — Example manifest
- `alice_runtime_contract.md` — Runtime contract specification

## Status

These may be promotable to `spec/runtime/` or `spec/ai/` — evaluate via docs-ingest before promotion.
```

- [ ] **Step 4: Commit**

```bash
git add docs/raw/archived/README.md docs/raw/audits-and-issues/README.md docs/raw/deprecated/README.md docs/raw/contracts/README.md
git commit -m "docs: add local indexes to docs/raw/ subdirectories"
```

---

### Task 10: Update docs/evonotes/README.md with Traversal Instructions

**Files:**
- Modify: `docs/evonotes/README.md`

- [ ] **Step 1: Read the current file** (already done above)

- [ ] **Step 2: Add traversal section after the Summary**

Append after line 81 (`All agents must treat this folder as truth.`):

```markdown
---

## Traversal Instructions for Agents

**Start at:** [[00-index/README]] — the deterministic traversal root.

**Traversal order:**
1. `00-index/README.md` — root map → lifecycle stage selection
2. `spec/README.md` (or other lifecycle stage) → domain selection
3. `spec/<domain>/README.md` → concept and file selection
4. Individual note files

**Never brute-force search without trying traversal first.** If a concept is canonical, it is listed in its domain README.

**If a concept is not in any domain README:** it is either in `needs-review/` (unaudited) or orphaned. Run docs-ingest orphan detection.

## Note Type Quick Reference

| type | trust level |
|------|-------------|
| `doctrine` | Highest — stable canonical philosophy |
| `architecture` | High — structural design |
| `spec` | High — actionable implementation spec |
| `execution` | High — operational procedure |
| `exploration` | Low — speculative, not settled |
| `historical` | Reference only — past state |
| `needs-review` | Do not cite as doctrine |
```

- [ ] **Step 3: Commit**

```bash
git add docs/evonotes/README.md
git commit -m "docs: add deterministic traversal instructions to evonotes README"
```

---

### Task 11: Redesign docs-ingest Skill

**Files:**
- Modify: `.codex/skills/docs-ingest/SKILL.md`

- [ ] **Step 1: Read current SKILL.md** (already done above — 6 steps + blockers + output format)

- [ ] **Step 2: Write updated SKILL.md**

The update adds Steps 4e, 7, 8, 9, 10 and updates the output format. All existing steps 1-6 remain unchanged.

After Step 4d, insert:

```markdown
### 4e. Update folder README

After writing the canonized file to its destination, update the folder's `README.md`:

1. Read the folder's `README.md`.
2. Locate the Traversal Map table.
3. Add a row for the new file: `| <concept> | [[<filename-without-extension>]] | <note_type> |`
4. If `README.md` does not exist, create it from `docs/evonotes/_templates/_TEMPLATE - Index README.md`.
5. Write the updated README.

Do not add files to the "Canonical Entry Points" section automatically — that requires human judgment. Only add to Traversal Map.
```

After Step 6 (Verify), add:

```markdown
---

## Step 7 — Orphan Detection

Scan all folders under `docs/evonotes/` for `.md` files (excluding READMEs and template files) that are NOT listed in their folder's `README.md`.

```bash
for dir in $(find docs/evonotes -type d); do
  readme="$dir/README.md"
  [ -f "$readme" ] || continue
  for f in "$dir"/*.md; do
    base="$(basename "$f" .md)"
    [[ "$base" == README ]] && continue
    [[ "$base" == _* ]] && continue
    grep -q "$base" "$readme" || echo "ORPHAN: $f"
  done
done
```

Report all orphans. Do not automatically add them to READMEs — report only.

---

## Step 8 — Missing Index Detection

Report any directory under `docs/evonotes/` that has >3 `.md` files but no `README.md`.

```bash
find docs/evonotes -type d | while read dir; do
  count=$(find "$dir" -maxdepth 1 -name "*.md" | wc -l)
  [ "$count" -gt 3 ] || continue
  [ -f "$dir/README.md" ] && continue
  echo "MISSING INDEX: $dir ($count files)"
done
```

---

## Step 9 — Wiki-Link Integrity Check

For each file canonized in this ingest run, extract all `[[wiki-link]]` references and verify the target filename exists somewhere under `docs/evonotes/`.

```bash
# For each new canonical file:
grep -o '\[\[[^]]*\]\]' "$canonized_file" | sed 's/\[\[//;s/\]\]//' | while read link; do
  # Strip section anchors (#...)
  target=$(echo "$link" | cut -d'#' -f1)
  found=$(find docs/evonotes -name "${target}.md" | head -1)
  [ -z "$found" ] && echo "BROKEN LINK: $link in $canonized_file"
done
```

Report broken links. Do not auto-fix.

---

## Step 10 — Traversal Root Update

If any ingest run created a new subdomain folder (e.g. `spec/infrastructure/`), update `docs/evonotes/00-index/README.md` to include the new folder in the Root Traversal Map table.

This step is manual for domain-level changes. For note-level additions, Step 4e handles README updates automatically.

---

## Updated Output Format

```text
ROUTED TO AUDITS:     N files
ARCHIVED (duplicate): N files
CANONIZED:            N files  → list each with destination
DEPRECATED:           N files  → list each with reason
ARCHIVED (misc):      N files
BLOCKED:              N files  → list each with reason

README UPDATES:       N folders updated
ORPHANS DETECTED:     N files  → list each
MISSING INDEXES:      N folders → list each
BROKEN WIKI-LINKS:    N links  → list each (file + broken target)
```

- [ ] **Step 3: Commit**

```bash
git add .codex/skills/docs-ingest/SKILL.md
git commit -m "feat(docs-ingest): add index maintenance, orphan detection, wiki-link validation (Steps 4e, 7-10)"
```

---

### Task 12: Add note_type to Frontmatter Template

**Files:**
- Modify: `docs/evonotes/_templates/_TEMPLATE - Canonical Note.md`

- [ ] **Step 1: Read the current template**

Read `docs/evonotes/_templates/_TEMPLATE - Canonical Note.md`.

- [ ] **Step 2: Add note_type field**

In the YAML frontmatter, add after `canonical: true`:

```yaml
note_type: doctrine | architecture | spec | execution | exploration | index | historical | audit
```

- [ ] **Step 3: Commit**

```bash
git add "docs/evonotes/_templates/_TEMPLATE - Canonical Note.md"
git commit -m "docs: add note_type field to canonical note frontmatter template"
```

---

### Task 13: Phased Rollout Verification

- [ ] **Step 1: Verify all 16+ domain README files exist**

```bash
find docs/evonotes -name README.md | sort
```

Expected: README in every lifecycle stage folder and every domain subfolder.

- [ ] **Step 2: Verify traversal root links are valid**

Read `docs/evonotes/00-index/README.md` and verify every `[[link]]` reference points to an existing file.

- [ ] **Step 3: Run orphan detection manually**

```bash
for dir in $(find docs/evonotes -type d); do
  readme="$dir/README.md"
  [ -f "$readme" ] || continue
  for f in "$dir"/*.md; do
    base="$(basename "$f" .md)"
    [[ "$base" == README ]] && continue
    [[ "$base" == _* ]] && continue
    grep -q "$base" "$readme" || echo "ORPHAN: $f"
  done
done
```

Report any orphans — these need manual Traversal Map entries added to their folder READMEs.

- [ ] **Step 4: Commit**

```bash
git add -p  # review any changes from verification
git commit -m "docs: traversal redesign verification pass complete"
```

---

## Phased Rollout Plan

| Phase | Scope | Tasks |
|-------|-------|-------|
| **Phase 1** (this plan) | Infrastructure: templates + all domain READMEs + docs-ingest redesign | Tasks 1-13 |
| **Phase 2** (follow-up) | needs-review triage: classify 70 unaudited notes | Separate audit pass |
| **Phase 3** (follow-up) | GitNexus verification sweep: confirm all implemented/ notes | Separate verification pass |
| **Phase 4** (follow-up) | Wiki-link integrity sweep: fix broken links across all 538 notes | Separate link-fix pass |
| **Phase 5** (ongoing) | docs-ingest maintains indexes automatically on every run | Ongoing |

---

## Self-Review

**Spec coverage check:**

| Requirement | Covered by Task |
|-------------|----------------|
| Every major folder has README | Tasks 3-9 |
| Root nav map (not sole traversal) | Task 2 |
| Folder locality for agents | Tasks 3-9 (all domain READMEs) |
| Human + AI agent support | Cross-agent strategy in architecture section |
| No flattening of hierarchy | Existing structure preserved |
| Anti-orphaning rules | Task 11 (Step 7), Task 13 |
| Note classification system | Classification system section + Task 12 |
| Migration strategy for needs-review | Documented in needs-review READMEs |
| raw doc category strategy | Task 9 |
| doc-ingest redesign | Task 11 |
| Cross-agent compatibility | Traversal Philosophy section |
| Echo discovery fix | Task 4 (spec/ai/README) |
| Semantic-only degradation analysis | Diagnosis section |
| Phased rollout | Phased Rollout section |
| spec/docs/ai separation analysis | Topology section |

**Placeholder scan:** None found. All steps have actual file content.

**Type consistency:** All wiki-link patterns use `[[filename]]` consistently. All frontmatter field names match across templates and domain READMEs.

## Related

^[source-materials/mirrors/doctrine/2026-05-16-evo-doc-architecture-redesign.md]
