---
title: EVOconnect — Delegator Talent Verification Doctrine
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVOconnect — Delegator Talent Verification Doctrine.md"]
updated: 2026-07-24
---

# EVOconnect — Delegator Talent Verification Doctrine

## Purpose

This raw doctrine note defines how the Delegator verifies Methods, Talents, Talent routes, variation routes, and resumable Method/Talent runs.

It extends the existing EVOconnect Talent, Method, Task Manager, and Delegator doctrine.

This note focuses on the execution safety contract between:

- Alice,
- the Delegator,
- the Task Manager,
- user-created Methods,
- user-created Talents,
- and user verification.

It does not replace existing Talent or Method doctrine. It clarifies how the Delegator enforces bounded execution without needing custom code for every Talent.

Related doctrine:

- [[EVOconnect — Lightweight Talent Structure Addendum]]
- [[MOC EVOconnect — Methods & Talents]]
- [[MOC EVOconnect — Delegator]]

---

## Core Principle

Delegator compliance does not prove reasoning correctness.

Delegator compliance proves bounded execution.

User verification proves whether bounded execution produced the intended result.

Methods and Talents do not make model behavior perfectly safe. They make model behavior bounded, auditable, interruptible, and correctable.

---

## Markdown Versus Manifest

User-created Methods and Talents have two different surfaces.

Markdown is for Alice and humans.

JSON manifest data is for the Delegator.

The markdown explains:

- what the workflow means,
- what Alice should understand,
- how Alice should approach each step,
- what the user should expect,
- and what deliverables the workflow is meant to produce.

The manifest defines:

- what route is allowed,
- which steps are allowed,
- which order those steps may run in,
- which tools each step may use,
- which transitions are allowed,
- which approvals are required,
- which outputs or checkpoints matter,
- and which deliverables complete the run.

Alice may read markdown.

The Delegator enforces the manifest.

Raw markdown is not execution authority.

---

## Manifest Role

The manifest is the enforceable execution contract.

It should be lightweight and deterministic.

It should not duplicate full markdown instructions unless export or portability requires it.

The manifest should reference package and step markdown files by path and hash.

The manifest should contain enough structure for the Delegator to answer:

- Is this workflow known?
- Is this version verified?
- Did the package source change?
- Did a step source change?
- Which route is active?
- Which step is current?
- Which tools are allowed right now?
- Which next steps are allowed?
- Did Alice skip a step?
- Did Alice add a step?
- Did Alice try to combine incompatible variations?
- Did Alice produce a required checkpoint or deliverable?
- Is user approval required before continuing?

---

## Package Markdown

A user-created Talent package may have a package-level markdown file.

This file is user-facing and Alice-facing.

It may include YAML frontmatter for broad metadata, such as:

```yaml
talent_id: talent.closeout.issue_set
version: 1.0.0
required_tools:
  - linear.read
  - github.read
  - gitnexus.query
permissions:
  external_tools: true
  user_confirmation_required: true
summary: >
  Closes out an implemented issue set by comparing Linear issues,
  GitNexus evidence, and planning specs.
deliverables:
  - closeout_report
  - gap_list
  - recommended_status_changes
```

The package markdown should summarize the Talent and link to step markdown files.

It should not be trusted by the Delegator without manifest validation.

---

## Step Markdown

Step markdown files are detailed instruction files for Alice and human inspection.

They may describe:

- the step goal,
- the reasoning pattern,
- edge cases,
- examples,
- user-facing wording,
- and interpretation guidance.

Step markdown is not allowed to grant tools by itself.

Tool permission comes from the manifest.

---

## Manifest Hashes

The manifest should include both package-level and step-level hashes.

The package hash answers:

- Did the Talent package as a whole change?

The step hash answers:

- Did this specific step instruction file change?

If a package or step hash changes, the verification state must be invalidated or reviewed according to policy.

This allows the Delegator to detect hidden edits without treating every unchanged step as newly untrusted.

---

## Required Manifest Shape

A manifest should include at minimum:

- workflow ID,
- version,
- package source path,
- package hash,
- package or source type,
- required tools,
- steps,
- routes,
- deliverables,
- and verification state.

Each step should include:

- step ID,
- step type,
- source path,
- source hash,
- allowed tools,
- expected output or checkpoint expectation when needed,
- and allowed transitions or route membership.

A simplified example:

```json
{
  "workflowId": "talent.closeout.issue_set",
  "version": "1.0.0",
  "packageSource": "talent.md",
  "packageHash": "sha256:...",
  "requiredTools": ["linear.read", "github.read", "gitnexus.query"],
  "steps": [
    {
      "stepId": "01",
      "stepType": "thinking",
      "source": "steps/01-load-issue-set.md",
      "sourceHash": "sha256:...",
      "allowedTools": []
    },
    {
      "stepId": "02",
      "stepType": "retrieval",
      "source": "steps/02-map-code-evidence.md",
      "sourceHash": "sha256:...",
      "allowedTools": ["github.read"]
    },
    {
      "stepId": "02a",
      "stepType": "retrieval",
      "source": "steps/02a-map-code-evidence-gitnexus.md",
      "sourceHash": "sha256:...",
      "variationOf": "02",
      "allowedTools": ["github.read", "gitnexus.query"]
    }
  ],
  "routes": [
    {
      "routeId": "default",
      "status": "trusted",
      "steps": ["01", "02", "03", "04", "05"]
    },
    {
      "routeId": "gitnexus-evidence",
      "status": "method",
      "variationOf": "default",
      "replaces": {
        "02": "02a"
      },
      "steps": ["01", "02a", "03", "04", "05"]
    }
  ],
  "deliverables": ["closeout_report", "gap_list", "recommended_status_changes"]
}
```

---

## Step Types

Step type tells the Delegator what kind of behavior is allowed.

Initial step types include:

- thinking,
- retrieval,
- execution,
- approval,
- verification,
- decision,
- and safe stop.

Thinking steps do not receive tool access by default.

Retrieval steps may read approved external context.

Execution steps may perform approved actions or tool calls.

Approval steps require user approval before continuing.

Verification steps collect user or system verification.

Decision steps select a declared route or transition.

Safe stop steps end or pause execution safely.

Rollback should not be a normal Talent step by default.

Rollback is a recovery action controlled by the Delegator, not a route Alice can freely choose.

---

## Tool Enforcement

Allowed tools are step-scoped.

If the current step is a thinking step and Alice tries to use a tool, the Delegator blocks the tool call.

If the current step allows only read tools and Alice tries to use a write tool, the Delegator blocks the action.

If Alice tries to use a tool not declared for the current step, the Delegator blocks the action.

The Delegator should then reprompt Alice with:

- the current route,
- the current step,
- the step type,
- the allowed tools,
- the denied behavior,
- and the expected next compliant behavior.

A first attempted violation is a correction event.

A repeated attempted violation after reprompt becomes a compliance failure.

---

## Route Enforcement

The Delegator validates declared routes, not improvised step combinations.

A route is a declared valid sequence.

Alice may not:

- skip a required step,
- add an undeclared step,
- run two steps at once,
- change route order,
- run a parent step and its variation together,
- or stack multiple variation steps inside a Talent variation route.

If the route declares:

```text
01 → 02 → 03 → 04 → 05
```

Alice cannot run:

```text
01 → 02 → 04 → 05
```

because step 03 was skipped.

Alice cannot run:

```text
01 → 02 → 02a → 03 → 04 → 05
```

because 02a replaces 02 and cannot run beside it.

Alice cannot run:

```text
01 → 02a → 03 → 04 → 05a
```

as a Talent variation route, because more than one variation step is active.

That route must become a new Method candidate if it is needed.

---

## Variation Enforcement

Canonical step IDs are zero-padded numbers:

```text
01
02
03
```

Variation step IDs are zero-padded numbers followed by one lowercase letter:

```text
02a
02b
05a
```

A variation step replaces its parent numeric step.

It does not supplement it.

Within a Talent variation route:

- at most one step ID may contain a variation suffix letter,
- the parent numeric step must not also appear,
- the variation step must declare which parent step it replaces,
- and the route must be explicitly declared in the manifest.

Valid routes:

```text
01, 02, 03, 04, 05
01, 02a, 03, 04, 05
01, 02, 03, 04, 05a
```

Invalid routes:

```text
01, 02, 02a, 03, 04, 05
01, 02a, 03, 04, 05a
```

This prevents combinatorial route expansion.

The Delegator should not validate every possible route combination.

It should validate only declared acceptable routes.

---

## Compliance Versus Correctness

The Delegator does not know whether Alice reasoned perfectly.

The Delegator knows whether Alice behaved within the manifest.

The Delegator can verify:

- current step,
- step order,
- route membership,
- tool scope,
- approval gates,
- output shape,
- checkpoint state,
- and allowed transitions.

The Delegator cannot fully prove:

- whether Alice understood the user perfectly,
- whether her comparison was semantically correct,
- whether the deliverable was actually useful,
- or whether the user’s intended outcome was satisfied.

Correctness is proven through:

- structured output validation,
- tests where available,
- user verification,
- and successful repeated runs.

---

## User Verification

User verification is what proves that bounded execution produced the intended result.

A run counts toward promotion only when:

```text
Delegator compliance passes
AND
the user confirms the outcome was successful.
```

If Alice follows the manifest but the user says the result is wrong, incomplete, or not useful, the run does not count toward promotion.

That is an outcome failure, not necessarily a safety failure.

---

## Failure Types

There are two major failure classes.

### Compliance Failure

A compliance failure happens when Alice cannot stay inside the manifest after corrective reprompt.

The ladder is:

1. Alice attempts invalid behavior.
2. Delegator blocks it.
3. Delegator reprompts Alice with the current route, current step, allowed tools, and expected behavior.
4. If Alice corrects herself, the run may continue and the correction is logged.
5. If Alice repeats the invalid behavior, Delegator shuts down the run.
6. User is notified.
7. Alice writes a journal or reflection entry.
8. Repeated failures may lead Alice to propose a revised Method version.
9. The revised Method re-enters approval and proving.

A first violation is a correction event.

A repeated violation after reprompt is a compliance failure.

### Outcome Failure

An outcome failure happens when Alice stays inside the manifest, but the user says the result was wrong, incomplete, or not useful.

Outcome failures reset promotion progress.

Outcome failures usually mean:

- the Method was experimental,
- the user did not inspect the proposal closely enough,
- Alice misunderstood the deliverable,
- the step instructions were too vague,
- or the expected output shape was valid but the content was wrong.

Both compliance failures and outcome failures prevent promotion.

Only compliance failure is automatically a safety concern.

---

## Correction Events

Correction events do not automatically break promotion.

If Alice attempts invalid behavior, Delegator blocks it, reprompts her, and she corrects herself, the run can still count if the final user verification passes.

The correction event should be logged.

Repeated correction events may indicate that the Method or Talent instructions are ambiguous and should be revised.

---

## Revision Loop

Failures do not promote trust.

Failures inform revision.

If failures repeat, Alice may propose a revised Method version using:

- failure logs,
- Delegator denial reasons,
- user feedback,
- and journal or reflection entries.

The revised Method is not trusted automatically.

It must re-enter approval and proving.

A revised route or manifest creates a new version.

---

## Immutable Trusted Routes

Promoted manifests and promoted routes are immutable.

Any change creates a new version.

Trusted execution contracts must not be edited in place.

If a package, step, route, tool permission, deliverable, or manifest hash changes, the system must treat the changed workflow as a new version requiring review according to policy.

---

## Task Manager Runtime State

The Task Manager is the runtime state ledger for Method and Talent runs.

The Delegator should always know:

- run ID,
- workflow ID,
- route ID,
- current step ID,
- completed steps,
- current checkpoint when present,
- allowed next steps,
- active tool grant,
- pending approval state,
- correction or failure count,
- and promotion streak status.

Method and Talent execution state belongs to the Task Manager, not the chat transcript.

Alice may resume only from Delegator-verified run state.

She may not infer execution state from chat memory alone.

---

## Checkpointed Inference Execution

Methods and Talents turn AI work from memory-heavy conversation into stateful, checkpointed execution.

A Method or Talent run may be broken into bounded inference segments.

A step may produce a checkpoint artifact when needed.

The system can then:

1. Load the current route, current step, and required context.
2. Perform the step.
3. Compress the result into a checkpoint artifact when needed.
4. Store the artifact in Task Manager run state.
5. Clear active model context or KV cache when useful.
6. Resume from the verified checkpoint on the next step.

This allows long-running workflows to continue without reloading the whole chat transcript.

It also allows weaker devices to perform heavier workflows by processing one bounded step at a time.

---

## Checkpoint Artifact Rules

Checkpoint artifacts are per-run artifacts.

They belong to Method or Talent run state in the Task Manager.

They are not automatically permanent doctrine or Talent package files.

Per-step checkpoint artifacts are optional, not mandatory.

Powerful devices may run multiple steps continuously without checkpoint compression.

Checkpoint artifacts are required or recommended when:

- resumability is needed,
- token pressure is high,
- the device is resource constrained,
- user approval pauses execution,
- the run must hand off to another instance or device,
- or a later step needs a verified artifact from an earlier step.

Alice resumes from Delegator-verified checkpoint artifacts when they exist.

She does not resume from conversational memory alone.

---

## Promotion Rule

A Method or Method-state route becomes eligible for Talent promotion only after successful user-confirmed runs.

A run counts only when:

- Delegator compliance passes,
- required approvals were honored,
- required deliverables were produced,
- and the user confirms the outcome was successful.

Outcome failures reset the promotion streak.

Compliance failures shut down the run after failed reprompt and require review or revision.

---

## Summary

Markdown guides Alice.

JSON constrains Alice.

Task Manager tracks state.

Delegator enforces boundaries.

User verification proves outcome.

Checkpoint artifacts enable resumability when needed.

Methods and Talents do not eliminate risk.

They reduce risk by separating reasoning guidance from execution authority.

## Related
