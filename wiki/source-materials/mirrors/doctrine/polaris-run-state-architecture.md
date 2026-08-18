---
title: polaris-run-state-architecture
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/polaris-run-state-architecture.md"]
updated: 2026-07-24
---

# EVO Precursor Planning Spec

## Run-State Architecture And Tracker-Agnostic Execution Telemetry

### Future Polaris Extraction Candidate

## Purpose

Define the next-generation EVO run-state architecture for workflow skills like evo-run, evo-plan, evo-analyze, evo-closeout, and docs-ingest.

This spec stabilizes the EVO-native precursor before any future Polaris extraction.

The goal is to make workflow execution:

- resumable,
- tracker-agnostic,
- auditable,
- low-context,
- low-token,
- deterministic,
- and portable across issue systems.

This is not Polaris yet.

Polaris would be the future generalized framework with templates, adapters, scaffolding, and reusable workflow patterns. This spec defines the EVO proving ground.

## Core Principle

Agents should not earn confidence by reading broadly.

Agents should earn confidence by following bounded routes, loading local doctrine, executing deterministic task chains, validating targeted assumptions, and checkpointing state.

## Problem

Current EVO skills rely mostly on mutable markdown run files, Linear-specific assumptions, and conversational continuity.

This works, but it creates problems:

- current-run files mix summary, state, telemetry, blockers, and history,
- Linear is baked too deeply into the workflow shape,
- agents may over-read repo context to feel safe,
- historical run traces are not cleanly separated from current state,
- steps are not first-class structured objects,
- and the current format is harder to generalize into Polaris.

## Required Direction

Split execution memory into four concepts:

1. Run
2. Step
3. Artifact
4. Telemetry

Each has a different job.

## Run

A run is the authoritative mutable snapshot of a workflow.

It answers:

“What is this workflow trying to accomplish, where is it pointed, and where should the agent resume?”

A run must include:

- run id
- title
- description
- skill name
- status
- tracker adapter
- target object
- branch/worktree info
- current step
- step array
- artifact references
- validation status
- timestamps
- resume cursor

The run is mutable and small.

The agent resumes from the run snapshot, not from replaying the full history.

## Tracker Adapter

The run must not be Linear-native.

It should support a generic tracker object.

Examples of possible trackers:

- Linear
- GitHub Issues
- Jira
- Beads
- local JSONL task files
- markdown task files

The run should store tracker info generically:

- tracker name
- tracker target id
- target type
- target url
- parent id if relevant
- child ids if relevant
- status mapping
- dependency mapping

So in EVO today, the tracker is Linear.

In Polaris later, Linear becomes only one adapter.

## Step

A step is a deterministic workflow unit.

It answers:

“What does this step prove, change, or decide?”

Each step should have:

- step id
- title
- description
- status
- order
- dependencies
- allowed skills/tools
- expected inputs
- expected outputs
- stop conditions
- validation requirements
- artifact references
- timestamps

Steps should be an array inside the run snapshot.

This matters because the workflow itself becomes a state machine, not just prose.

## Artifact

An artifact is an output or evidence object created during the run.

Examples:

- analysis report
- final summary
- validation result
- GitNexus result
- PR body draft
- doctrine note
- implementation diff summary

Artifacts should be referenced by id/path from the run and step objects.

Artifacts may be markdown, JSON, or other files depending on purpose.

## Telemetry

Telemetry is append-only operational history.

Preferred format: JSONL.

Telemetry answers:

“What actually happened during the run?”

It should record events like:

- run started
- step started
- step completed
- file read
- tool invoked
- validation passed
- blocker found
- branch created
- commit created
- PR opened
- tracker updated
- run completed

Telemetry should not be used as the primary resume source.

It is for replay, audit, analytics, debugging, and future operational learning.

## Snapshot vs Telemetry

The snapshot is mutable.

The telemetry log is append-only.

The snapshot is for fast resume.

The telemetry log is for history.

Do not force agents to replay thousands of telemetry events just to know what to do next.

## Human Readability

Markdown still matters.

The system should keep human-readable summaries for:

- reviews,
- handoffs,
- debugging,
- final reports,
- PR bodies,
- and audits.

But markdown should not carry all machine state forever.

Markdown is the human layer.

JSON/JSONL is the operational layer.

## Proposed File Shape

For each workflow skill, the future shape could be:

- current-run.md
- current-state.json
- runs/
- summaries/
- artifacts/

Where:

- current-run.md is the human-readable current summary,
- current-state.json is the mutable machine snapshot,
- runs/*.jsonl is append-only telemetry,
- summaries/*.md is completed human-readable run history,
- artifacts/ stores supporting evidence.

## Example Run Shape

```json
{
  "run_id": "run-2026-05-21-001",
  "title": "Run EVOS1-345 avatar cleanup analysis",
  "description": "Analyze Alice avatar asset ownership, verify prior cleanup claims, and produce implementation clusters.",
  "skill": "evo-analyze",
  "status": "running",
  "tracker": {
    "name": "linear",
    "target_type": "issue",
    "target_id": "EVOS1-345",
    "target_url": "https://linear.app/...",
    "parent_id": null,
    "child_ids": [],
    "status": "In Progress"
  },
  "branch": null,
  "worktree": null,
  "current_step_id": "03-assess-issue",
  "resume_cursor": 12,
  "steps": [
    {
      "step_id": "01-fetch-and-orient",
      "title": "Fetch and orient",
      "description": "Load the tracker target, confirm scope, inspect routing, and check graph freshness.",
      "status": "complete",
      "order": 1,
      "dependencies": [],
      "allowed_skills": ["caveman", "gitnexus"],
      "expected_outputs": ["tracker target loaded", "routing scope confirmed"],
      "stop_conditions": ["target missing", "scope conflict"],
      "validation": {
        "status": "passed",
        "checks": ["tracker target fetched", "repo route identified"]
      },
      "artifacts": [],
      "started_at": "2026-05-21T10:00:00Z",
      "completed_at": "2026-05-21T10:02:00Z"
    }
  ],
  "artifacts": [],
  "validation_status": "in_progress",
  "created_at": "2026-05-21T10:00:00Z",
  "updated_at": "2026-05-21T10:05:00Z",
  "completed_at": null
}
```

## Example Telemetry Events

```json
{"event":"run-start","run_id":"run-2026-05-21-001","skill":"evo-analyze"}
{"event":"tracker-target-loaded","tracker":"linear","target_id":"EVOS1-345"}
{"event":"step-start","step_id":"01-fetch-and-orient"}
{"event":"validation-pass","step_id":"01-fetch-and-orient","check":"repo route identified"}
{"event":"step-complete","step_id":"01-fetch-and-orient"}
{"event":"step-start","step_id":"02-map-affected-code"}
```

## Over-Reading Control

This architecture should reduce over-reading.

Agents should not inspect broadly to feel safe.

Each step should define:

- allowed files,
- allowed routes,
- allowed skills,
- expected evidence,
- and validation boundaries.

If the step cannot justify a file read through routing, graph impact, or explicit scope, it should not read it.

## Codex-Specific Concern

Codex appears powerful but prone to reading more context than necessary during implementation-heavy work.

This may be model behavior, not just skill behavior.

The run-state architecture should constrain this by making each step declare:

- what it needs,
- why it needs it,
- what evidence is sufficient,
- and when it must stop reading.

## Success Criteria

This architecture succeeds if:

- agents resume from compact state,
- runs become tracker-agnostic,
- steps become deterministic workflow units,
- old telemetry remains available without bloating current context,
- agents read fewer irrelevant files,
- audits remain human-readable,
- and future Polaris extraction becomes straightforward.

## Non-Goals

This spec does not:

- build Polaris,
- replace Linear,
- replace GitHub Issues,
- replace Beads,
- replace SpecKit,
- replace GitNexus,
- or remove markdown summaries.

It creates the EVO-native precursor pattern that Polaris may later generalize.

## Related

^[{src_rel}]
