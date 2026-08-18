---
title: 2026-06-10-genesis-agent-doctrine
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/2026-06-10-genesis-agent-doctrine.md"]
updated: 2026-07-24
---

# CLAUDE.md

# git-fit Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-01-27

## Spec-Kit Location

**Central Spec-Kit**: `.github/spec-kit/`

- **Constitution**: `.github/spec-kit/memory/constitution.md` (v1.0.0)
- **Templates**: `.github/spec-kit/templates/` (git-fit customized)
- **Scripts**: `.github/spec-kit/scripts/` (automation tools)

## Routing Reference

Claude must load `.evo/routing.md` before analysis or planning in any folder.

Claude must load the nearest `INSTRUCTIONS.md` before assessing changes in that
folder.

Claude's planner/analyst posture is defined in `AGENTS.md`; routing applies
equally to Claude.

If folder instructions conflict with issue text or doctrine, Claude must surface
the conflict before proposing a plan.

## Active Technologies

- **Stack**: TypeScript 5.0+, JavaScript ES2022, SvelteKit 2.22+, Supabase 2.46+, Capacitor 7.4+, Tailwind CSS 4.1+
- **Mobile**: Flutter (Dart), Swift 5.9+, Kotlin 1.9+
- **AI/ML**: Llama 3.1 8B (4-bit GGUF), llama.cpp (Metal/NDK), on-device inference
- **Voice**: Kokoro TTS (on-device), Supertonic TTS (on-device)
- **Database**: Supabase PostgreSQL with real-time subscriptions, IndexedDB for offline
- **Deployment**: Vercel (web), TestFlight (iOS)

## Project Structure

```
src/
tests/
```

## Commands

npm test; npm run lint

## Code Style

TypeScript 5.0+, JavaScript ES2022: Follow standard conventions

## Recent Changes

- Migrated from Convex to Supabase (2025-01-27)
- Added on-device AI with Llama 3.1 8B for Alice
- Integrated Kokoro and Supertonic TTS for voice synthesis
- Flutter app handles all Alice interactions (web not used for Alice)

## EVO Architecture Anti-Drift Rules

- `LoRAKind = { U, T, GU, GT }`; ENF and VOICE adapters are removed.
- Current safety architecture is two-layer: `GatingEngine` deterministic enforcement plus `AnswerRepair` domain-aware UX repair.
- Do not reintroduce an ENF safety layer or three-layer defense framing.
- MLX was evaluated and abandoned; current mobile inference runtime is GGUF plus llama.cpp on iOS and Android.
- Phi-4-mini is replaced by Qwen2.5-1.5B Q4_K_M.
- ElevenLabs TTS is replaced by Supertonic 2 on-device ONNX.
- Treat `docs/raw/` notes that recommend ENF, VOICE, or MLX as historical until explicit deprecation resolution exists.
- The `connect` domain package for inter-app contracts is distinct from the EVOconnect product.

## Docs Ingestion Rule

When new files are added to `docs/raw/`, always process them with the ingestion skill before touching anything manually:

```
Use docs-ingest.
```

Skill: `.codex/skills/docs-ingest/SKILL.md` (with adapter/reference-compatible surfaces at `.agents/skills/docs-ingest/SKILL.md` and `.claude/skills/docs-ingest/SKILL.md`)

- Linear issue files (`EVOS1-*`, `EVOC-*`, `EVOTRA-*`, etc.) → `docs/raw/audits-and-issues/`
- Files already in `docs/EVOnotes/` by filename → `docs/raw/archived/`
- New doctrine files → formatted with canonical frontmatter → `docs/EVOnotes/spec/[domain]/` → raw original archived
- Files referencing ENF, VOICE, MLX, Phi-4-mini, or ElevenLabs → `docs/raw/deprecated/`
- Misc unclassified files → `docs/raw/archived/`
- Every run performs a backlink sweep of ALL notes in `implemented/` and `needs-review/` — unlinked notes get inbound links added from related notes, and reciprocal links added back; this ensures the graph is fully connected for evo-plan and evo-run traversal

Do not manually promote raw files into `docs/EVOnotes/` without running the ingestion skill first.

## EVO Cluster Review Doctrine

Claude doctrine defines review and governance behavior.
Reusable operational workflows belong in `.codex/skills/`; `.agents/skills/` is an adapter/reference-compatible surface and should not be treated as source of truth.

Claude's default role in governed EVO clusters is review, sanity checking, and
architecture auditing. Claude may implement only when the active prompt or
cluster child explicitly assigns implementation work.

When reviewing a governed EVO cluster, Claude must:

- treat the Linear parent issue as the cluster contract and PR container,
- verify that child execution followed the governed execution loop,
- verify that only the current lowest-numbered open child was executed at any time,
- verify that completed children caused the workflow to re-fetch and continue to the next open child,
- verify that blocked children stop forward execution,
- verify that all child work stayed within parent scope,
- verify that adjacent discoveries became follow-up issues instead of silent scope expansion,
- verify that one branch and one draft PR represent the cluster,
- verify that the PR targets `testing` unless the parent explicitly says otherwise.

## GitNexus Review Expectations

Claude must use GitNexus as a review and wiring sanity-check tool when it is
available. If the GitNexus index is stale, Claude must report the staleness and
combine targeted GitNexus results with direct repository inspection instead of
assuming the index is current.

For doctrine or wiring reviews, Claude should check:

- relevant architecture and instruction files before assessing changes,
- changed symbols or files with GitNexus impact or change detection,
- whether affected execution flows match the claimed scope,
- whether generated GitNexus blocks conflict with durable manual instructions,
- whether stale-index warnings change confidence in the review.

## Workflow Skills Review

Claude should treat repository-local skills as operational workflow units rather than durable doctrine.

When reviewing `.codex/skills/` and adapter/reference-compatible surfaces under `.agents/skills/`:

- verify valid YAML frontmatter exists,
- verify the workflow does not conflict with `AGENTS.md`,
- verify blocker handling remains deterministic,
- verify PR behavior matches cluster doctrine,
- verify the workflow reduces orchestration drift and prompt repetition,
- verify skills do not silently grant unsafe autonomy.

Claude should prefer concise workflow skills over large duplicated instruction blocks.

## Wiring And Architecture Sanity Pass

Before approving or summarizing cluster work, Claude must perform a sanity pass
for instruction consistency and doctrine drift:

- branch and PR rules must match the parent contract,
- blocker handling must be deterministic and auditable,
- Codex, Claude, skills, and workflow doctrine must remain complementary rather than contradictory,
- runtime or app code must not change in documentation-only clusters,
- generated or deprecated instruction files must not become the source of new
  doctrine unless explicitly requested.

Claude should report findings first when reviewing, then summarize validation,
residual risk, and any follow-up issues.

<!-- MANUAL ADDITIONS START -->
## Validation And Handoff

When reviewing governed cluster work, Claude should summarize:

- what changed,
- what was validated,
- residual risks,
- stale-index concerns,
- follow-up issues,
- and whether the PR is safe to merge.

Claude should report findings first and recommendations second.

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **git-fit** (58223 symbols, 109176 relationships, 300 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/git-fit/context` | Codebase overview, check index freshness |
| `gitnexus://repo/git-fit/clusters` | All functional areas |
| `gitnexus://repo/git-fit/processes` | All execution flows |
| `gitnexus://repo/git-fit/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->
<!-- MANUAL ADDITIONS END -->

---

# AGENTS.md

# Agent Instructions

This repository uses Linear as the primary execution tracker.

## Source of Truth

- Linear = execution truth
- GitHub = delivery truth
- Repository doctrine files and skills = operational truth
- Optional tools (`bd`, beads, etc.) must never block completion unless explicitly required

## Routing Traversal

Before acting on any issue, Worker agents must load `.evo/routing.md`.

Before assessing changes in a folder, find and load the nearest `INSTRUCTIONS.md`
relative to the folder being edited.

If folder instructions or doctrine conflict with the issue, stop and surface the
conflict before implementing, using the conflict report format defined in
`.evo/routing.md`.

Do not skip directly from issue text to implementation when folder instructions
may change the correct behavior.

## Raw Note Review - Stale Reference Guard

Treat `docs/raw/` notes that reference ENF, VOICE, MLX-as-recommendation, Phi-4-mini, or ElevenLabs TTS as historical until explicit deprecation resolution exists. Do not promote those notes into `docs/EVOnotes/` without resolving stale references against current doctrine: `LoRAKind = { U, T, GU, GT }`, safety is `GatingEngine` plus `AnswerRepair`, mobile inference is GGUF plus llama.cpp, the current small model is Qwen2.5-1.5B Q4_K_M, and on-device TTS is Supertonic 2.

## EVO Workflow Doctrine

This repository uses governed EVO cluster execution.

When assigned a Linear parent cluster:

- the parent issue is the cluster contract and PR container
- one cluster = one branch
- one cluster = one PR
- target branch defaults to `testing` unless overridden by the parent
- use or create the parent-scoped branch named in the parent
- execute children in numeric order only
- execute only the lowest-numbered open child at any time
- after completing a child, re-fetch children and continue to the next lowest-numbered open child
- stop immediately if a child is blocked
- do not skip blocked children
- do not perform opportunistic cleanup
- do not perform unrelated refactors
- do not make cross-cluster changes unless explicitly approved
- adjacent discoveries must become follow-up Linear issues

## Child Execution Rules

For each child issue:

1. Move the child to In Progress
2. Perform only the scoped child work
3. Use GitNexus and direct repository inspection as needed
4. Run the narrowest useful validation
5. Commit if files changed
6. Add concise evidence comments
7. Mark the child Done only after acceptance criteria pass
8. Re-fetch children and continue execution

## PR Rules

After all children are Done:

- run final targeted validation
- push the parent-scoped branch
- create exactly one draft GitHub PR targeting the required base branch
- include:
  - parent issue ID
  - completed child issues
  - validation performed
  - risks
  - follow-up issues

Do not push or create a PR if any child is blocked, incomplete, or awaiting approval.

## Blocker Protocol

If blocked:

- stop execution immediately
- create appropriate `blocked by` or `blocks` Linear relationships when applicable
- add a Linear comment with the unblock condition
- apply the `blocked` label if available
- do not continue to later child issues
- do not push
- do not create a PR

Do not assume a `Blocked` workflow status exists.

## Workflow Skills

Canonical runnable shared workflow skills for Codex live under:

`.codex/skills/`

`.agents/skills/` is an adapter/reference surface for harness compatibility.

Skills provide reusable operational workflows.
Doctrine files define governance and behavioral rules.

Use skills to:

- reduce prompt repetition
- reduce orchestration drift
- reduce token usage
- standardize execution behavior

Prefer lean execution over excessive decomposition.

## Docs Ingestion

When new files are dropped into `docs/raw/`, use the ingestion skill to triage them:

```
Use docs-ingest.
```

Skill: `.codex/skills/docs-ingest/SKILL.md` (`.agents/skills/docs-ingest/SKILL.md` remains adapter/reference compatible)

The skill:
1. Routes Linear issue files to `docs/raw/audits-and-issues/`
2. Archives raw files that already have a canonical counterpart in `docs/EVOnotes/`
3. Canonizes new doctrine files into `docs/EVOnotes/spec/` with correct frontmatter
4. Moves stale-reference files (ENF, VOICE, MLX, Phi-4-mini, ElevenLabs) to `docs/raw/deprecated/`
5. Archives remaining unclassified files

Do not manually move or promote raw files without running this skill first.
Do not promote raw files that reference deprecated systems without resolving stale references.

## Caveman Usage

Caveman-style decomposition and compression may be used selectively when it reduces reasoning overhead.

Use Caveman when:

- decomposing large implementation tasks
- compressing accumulated reasoning
- reducing repeated repo analysis
- narrowing execution scope

Do not use Caveman when:

- the task is already narrow and deterministic
- decomposition adds unnecessary orchestration overhead
- direct execution is cheaper than delegation

## GitNexus Rules

Use GitNexus as targeted code intelligence, not broad repo analysis.

Preferred workflow:

- use targeted GitNexus queries before editing
- inspect only relevant files and symbols
- use direct inspection for narrow tasks
- use broad reindexing only when the index is stale and the task requires it

Before modifying significant symbols:

- run impact analysis
- warn the user if impact risk is HIGH or CRITICAL

Before committing:

- run change detection to verify affected scope

## Completion Rules

Work is not complete unless:

- required changes are committed
- changes are pushed
- and a real GitHub PR exists when required

Never claim completion without a real PR URL when a PR is required.

## Environment Failure Rules

If the environment cannot complete required work:

- report blocked by environment
- state the exact failing step
- include the failed command/tool/capability
- stop short of claiming completion

## Validation Rules

If code changed:

- run the most relevant available validation
- report what could not be validated and why

## Run Lineage

Every EVO skill invocation generates a unique `run_id` at session start. The `run_id` must appear in:

- `current-state.json` (top-level field)
- `current-run.md` (first field, after the heading)
- JSONL telemetry (every event)
- Linear evidence comments (opening lines of every comment)
- PR body footer (required when a PR is created)

When a session is resumed or an issue is reopened, set `related_run_id` to the prior `run_id`. When a run is spawned by a parent orchestration flow, set `parent_run_id`.

See `.evo/run-state/lineage-governance.md` for complete lineage, lifecycle, and handoff rules.

## Handoff Format

At the end of every session provide:

- `run_id` of this session
- what changed
- what was validated
- what remains
- whether changes were pushed
- whether a real PR was created
- any blockers
- resume command (if applicable): `Use <skill> on <issue-id>. Continue from step <N>. run_id: <run_id>.`

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **git-fit** (58223 symbols, 109176 relationships, 300 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/git-fit/context` | Codebase overview, check index freshness |
| `gitnexus://repo/git-fit/clusters` | All functional areas |
| `gitnexus://repo/git-fit/processes` | All execution flows |
| `gitnexus://repo/git-fit/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->

## Related

^[{src_rel}]
