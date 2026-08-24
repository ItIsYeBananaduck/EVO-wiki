---
title: Purpose
type: concept
tags: [connect, evo]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# Purpose

Define how EVOconnect can let Alice safely operate desktop apps and files when no API, MCP server, connector, Shortcut, terminal path, or browser-safe adapter exists.

The goal is not blanket computer control. The goal is a permission-mapped workspace where Alice can learn, repeat, and automate approved work without gaining unrestricted access.

# Core Doctrine

Alice may observe broadly only when approved.

Alice may act narrowly by default.

Alice may automate only known, scoped, validated workflows.

Alice does not have generic computer access. Alice has mapped access.

# Execution Priority

EVOconnect should prefer structured execution before UI control.

1. Native API
2. MCP server
3. Connector
4. Apple Shortcuts / system automation
5. Terminal adapter
6. Browser adapter
7. Scoped Desktop Control Adapter

The Scoped Desktop Control Adapter is a fallback. It is powerful, but it is more fragile than structured APIs because buttons move, app layouts change, and modal states can interrupt workflows.

# Scoped Desktop Control Adapter

The Scoped Desktop Control Adapter allows Alice to operate a desktop app through screen observation and controlled UI actions.

It should be used when the target system does not expose a safer structured integration path.

This adapter should always be governed by the Delegator.

The Delegator controls:

- app scope
- window or file scope
- action scope
- time scope
- read/write level
- user approval requirements
- audit logging
- validation criteria

# Permission Modes

## Live Approval

Used when the user is present and Alice needs access to an app or surface during the session.

Example:

> I would like to access Figma for this session to inspect the current EVOtraining design file. Approve?

## Deferred Approval

Used when Alice finds possible work but the user is not actively approving every step.

Example:

> I found 3 cleanup actions I can perform. Approve all?

## Pre-Approved Talent Permission

Used only for known workflows that have already been performed, validated, and approved for unattended execution.

Example:

> Alice may open Finder every Friday and move receipt PDFs from Downloads into Finance/Receipts when filename and content match receipt criteria.

Pre-approval applies to a specific Method or Talent, not to an entire app.

Not allowed:

> Alice can use Finder whenever.

Allowed:

> Alice can move receipt PDFs from Downloads to Finance/Receipts for the Receipt Filing Talent.

# Unattended Execution Rule

Alice may run workflows while the user is away only when the workflow is:

8. previously performed
9. validated
10. promoted or approved as a Method/Talent
11. pre-approved by the user
12. bound to a narrow scope
13. logged and reviewable after execution

If the workflow is new, ambiguous, destructive, or outside scope, Alice must ask first.

This allows routine desktop work to happen automatically without turning Alice into a free-clicking agent.

# Method to Talent Flow

## First Run

Alice asks permission.

The Delegator records:

- user intent
- app used
- window/file context
- permission scope
- observation summaries
- keyframes when enabled
- action sequence
- user approvals
- outcome
- validation result

## Repeated Runs

The workflow becomes a Method.

Alice can suggest:

> I have done this before. Use the same method?

## Talent Promotion

After repeated successful runs, the Delegator may propose:

> This workflow has succeeded 3 times. Promote it to a Talent?

Talent promotion should require:

- same goal
- same app or app family
- predictable state transitions
- successful validation
- no unsafe side effects
- user approval

# Screen Recording Doctrine

Screen recording is perception, not memory.

Default behavior:

- no stored video
- no continuous recording archive
- temporary frames only during execution
- structured action logs retained

Optional workflow-learning mode:

- store keyframes only
- local only by default
- encrypted
- tied to Method/Talent records
- user reviewable
- user deletable
- not used for general model training unless separately approved

Keyframes can be useful for Talent training because they help replay and validate workflows while reducing future prompt cost.

Store:

- initial state
- before critical action
- after critical action
- error state
- final success state

Do not store continuous video.

Best principle:

> Store meaning, not pixels.

# UI Interaction Record

Each UI action should generate a structured record.

```json
{
  "interaction_id": "uuid",
  "task_id": "connect_task_id",
  "method_id": "optional_method_id",
  "talent_id": "optional_talent_id",
  "app": "Figma",
  "window_context": "EVOtraining UI file",
  "permission_scope": {
    "duration": "session",
    "access": "read_write",
    "allowed_actions": ["inspect", "click", "type", "edit_current_file"],
    "forbidden_actions": ["delete_file", "share_publicly", "change_billing"]
  },
  "observation": {
    "type": "screen_frame_analysis",
    "stored_raw_frame": false,
    "summary": "Settings panel is open. Save button visible."
  },
  "action": {
    "type": "click",
    "target": "Save button",
    "reason": "Apply approved UI change"
  },
  "result": {
    "status": "success",
    "validation": "Save confirmation appeared"
  },
  "requires_user_review": false
}
```

# Safety Boundaries

The adapter must never have unrestricted machine control.

Hard limits:

- no password entry unless the user manually handles it
- no financial transactions without explicit approval
- no deleting files without explicit approval
- no sending messages or emails without explicit approval
- no installing software without explicit approval
- no changing security or privacy settings without explicit approval
- no broad app access as a pre-approved Talent

# EVOconnect Finder

EVOconnect needs its own Finder-like interface.

This is not only a file browser. It is Alice’s visible permission map.

The user can see and access the whole system through the Connect UI.

Alice can only see and act on the parts of the system that are open to her.

# Open Zones and Bunkers

## Open Zones

Files, folders, apps, or workspaces Alice can inspect or use under approved rules.

Examples:

- EVO project folder
- Downloads/Receipts
- Figma design export folder
- Linear export folder

## Bunkers

Files, folders, apps, or data Alice cannot access unless the user explicitly opens them.

Examples:

- taxes
- legal documents
- medical files
- private photos
- relationship notes
- financial accounts
- credentials

The user can still access bunkered files normally inside Connect.

Alice cannot.

This is permission geometry, not just hiding files.

# Access Levels

Each file, folder, app, or workspace can have an access level.

- Hidden from Alice
- Visible metadata only
- Read-only
- Suggest changes
- Edit with approval
- Edit automatically for a specific Talent
- Never access

# Bunker Doctrine

Open zones are fair game only within their assigned permission rules.

Bunkers are off-limits to Alice by default.

The user owns the whole workspace.

Alice receives a governed map of the workspace.

# Product Principle

EVOconnect turns the computer into a permission-mapped workspace where Alice can safely learn, repeat, and automate approved work without gaining blanket control.

This makes routine automation possible while preserving trust, visibility, and user control.

# Related Implementation Implications

This concept likely requires future Linear issues for:

- Scoped Desktop Control Adapter
- session-scoped permission grants
- pre-approved Talent execution policy
- screen keyframe capture and retention settings
- UI interaction record schema
- EVOconnect Finder permission map
- Open Zone / Bunker access controls
- unattended workflow audit log
- Talent replay validation

## Related
- [[EVOconnect — System Map]]
- [[EVO Architecture Bible]]
- [[EVOconnect — Action Bar & Mini Action Bar System.md]]
- [[EVOconnect — Coach Pane Pack Contract.md]]
- [[EVOconnect — Connect Library & Unified Access Layer.md]]
- [[EVOconnect — Hive Node Architecture.md]]
- [[EVOconnect — Lightweight Talent Structure Addendum.md]]
- [[EVOconnect — Method Reconstruction Model.md]]
^[wiki-native — no upstream source]
