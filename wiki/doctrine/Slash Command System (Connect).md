---
title: Slash Command System (Connect)
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Slash Command System (Connect).md
updated: 2026-07-24
---

# Slash Command System (Connect)
## Purpose

Define the slash command system for EVOconnect.

Slash commands give power users a fast way to intentionally trigger Alice talents from chat without bypassing governance, approval, or domain routing.

## Core Principle

**Slash commands trigger governed talents. They do not directly mutate system state.**

A slash command is an explicit user intent signal. It can start a workflow, draft, task, or proposal, but the underlying talent still applies all required validation, approval, routing, and trace rules.

## Scope

This note defines Connect slash commands.

Connect slash commands may reference other domains, but the command layer itself belongs to EVOconnect because Connect is the cross-domain coordination surface.

## Command Format

Basic format:

```plain text
/command optional arguments
```

Examples:

```plain text
/note
/note this
/note Living Notes should require approval before creation.
/task create follow-up issue for the Living Notes talent
/decision capture this as a system decision
```

## Initial Required Command

### /note

Triggers the Create Living Note Talent.

`/note` starts the draft stage. It does not directly create a Living Note.

Supported forms:

```plain text
/note
```

Uses recent chat context.

```plain text
/note this
```

Uses the immediately relevant prior message or selected context.

```plain text
/note <inline text>
```

Uses the inline text as the primary note seed.

## /note Behavior

When `/note` is invoked, Alice must:

1. gather the smallest useful context bundle
2. classify the candidate note
3. check for existing related notes
4. create a Living Note draft
5. propose links to notes, projects, phases, tasks, maps, talents, or workflows
6. show the draft for user review
7. wait for approval
8. create or update the Living Note only after approval
9. mark created or updated notes as Not Synced
10. index and map the note after approval

## Future Commands

The system should be designed to support additional commands without changing the core governance model.

### /task

Creates a governed task draft or Linear-ready issue proposal.

May link to:

- project
- phase
- parent task
- originating note
- relevant workflow

### /decision

Captures a decision draft.

Useful for architecture, doctrine, product, and governance decisions.

May become:

- Living Note
- decision note
- update to an existing note

### /workflow

Starts a workflow draft or workflow execution proposal.

Must route through the relevant workflow talent and Delegator rules.

### /link

Proposes links between existing notes, projects, phases, tasks, talents, or workflows.

Does not silently create backlinks without user approval when the link changes trusted knowledge.

### /scratch

Creates or updates a scratch-style temporary note when the user wants lightweight capture without making trusted Living Notes.

### /project

Creates a project draft or links current context to an existing project.

### /phase

Creates or links a phase context.

### /help

Shows available slash commands and their purpose.

## Command Routing

All commands route through a command registry.

```plain text
Slash input
→ parse command
→ resolve command intent
→ select governed talent
→ gather minimal context
→ run talent preflight
→ draft/propose/action according to talent rules
→ approval gate if required
→ execute or create after approval
→ trace result
```

## Command Registry Requirements

Each command should define:

- command name
- aliases, if any
- owning domain
- owning talent
- required permissions
- whether approval is required
- expected input shape
- output type
- trace requirements
- failure behavior

Example registry entry:

```plain text
Command: /note
Owner: EVOconnect
Talent: Create Living Note Talent
Approval Required: Yes, before creation
Output: Living Note draft or update proposal
Trace: source context, proposed links, approval result
```

## Governance Rules

- Slash commands must not bypass talents.
- Slash commands must not bypass Delegator rules.
- Slash commands must not silently mutate trusted knowledge.
- Commands that create, update, link, delete, or execute require the relevant approval model.
- `/note` creates drafts, not notes.
- Commands must preserve traceability.
- If a command belongs to another domain, Connect routes it rather than absorbing domain knowledge.

## Domain Routing

Connect may receive commands that target another domain.

Example:

```plain text
/note this training fatigue rule
```

If the content is pure Training knowledge, Alice should route it to Training cognition rather than creating a Connect Living Note.

If the content is a cross-domain workflow rule involving Training, then Connect may create a Living Note that references Training cognition.

## UX Behavior

Slash commands should feel fast, not heavy.

Alice should respond with compact drafts and clear next actions.

For example:

```plain text
I can draft that as a Living Note.
Proposed links: EVO Cognition Layer, Living Notes — Connect Knowledge System.
Approve, revise, or cancel?
```

Power users should be able to invoke commands without leaving chat.

## Error Handling

### Unknown command

Alice should explain the command is unknown and suggest `/help`.

### Ambiguous command

Alice should ask the smallest useful clarification.

### Missing context

Alice should request the missing context or draft with uncertainty clearly marked.

### Domain mismatch

Alice should route the command to the correct domain or explain why Connect should not own it.

### Approval declined

Alice should cancel without mutating trusted knowledge.

## Relationship To Living Notes

Slash commands provide an explicit trigger path for Living Note creation.

They complement passive detection by letting the user say:

> capture this now

without relying on Alice to decide whether something is important enough to suggest.

## Relationship To Talents

Every command should map to a talent or governed workflow.

Slash commands are not raw function calls. They are user-friendly entry points into approved execution paths.

## Relationship To Delegator

Delegator remains responsible for enforcing permissions, approvals, tool access, domain boundaries, and safety constraints.

Slash commands express intent. Delegator governs action.

## Related Notes

- [EVO Cognition Layer](https://www.notion.so/354c72bad01381aeb193f1e9b2bd2b1f)
- [Living Notes — Connect Knowledge System](https://www.notion.so/354c72bad013815bb055d0e1f13816fa)
- [Living Note Creation Flow](https://www.notion.so/354c72bad01381668eadf9d85c083a1d)
- [Create Living Note Talent](https://www.notion.so/354c72bad013816da597eeb0afbd3ebd)
- [Token-Light Knowledge Navigation](https://www.notion.so/354c72bad013815cb880ce154948e879)
- Domain Maps & Knowledge Maps

## Summary

The Connect slash command system gives power users fast, explicit control while preserving EVO governance.

`/note` is the first required command. It starts the Create Living Note Talent and produces an approved Living Note only after draft review and user approval.
^[source-materials/mirrors/doctrine/Slash Command System (Connect).md]
