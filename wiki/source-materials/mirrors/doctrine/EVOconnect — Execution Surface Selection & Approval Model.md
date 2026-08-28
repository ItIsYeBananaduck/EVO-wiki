---
title: "Execution Hierarchy"
type: doctrine
tags: ['lsctech', 'doctrine', 'source-material', 'evo']
updated: 2026-08-24
---



## Core Principle

> Alice should attempt the most structured path first, adapt based on success and user preference, and operate transparently with flexible approval controls.

---

# Execution Hierarchy

## Default Order

1. Plugin (structured provider access)
2. EVObrowser (UI-based fallback)
3. EVOterminal (local/system execution)
4. Manual fallback (user-assisted)

---

## Plugin vs Browser Rule

### Default Behavior
- Always attempt plugin first if available

### Fallback Behavior
- If plugin:
  - fails
  - is incomplete
  - produces poor results

→ Automatically fallback to EVObrowser

---

## User Override

> User can override selection at any time

Examples:
- “Use the browser for this”
- “Don’t use the plugin”
- “Run this locally”

Alice should:
- respect immediately
- remember preference
- apply it to future similar tasks

---

# Learning Behavior

## Preference Memory

Alice should learn:

- per service (QuickBooks, vendor portal, etc.)
- per task type (report download, update record, etc.)
- per user preference

### Example

- QuickBooks → plugin preferred
- Vendor X → browser preferred
- Local scripts → terminal preferred

---

## Adaptive Routing

Over time, selection becomes:

> best-known-success-path

Not just static hierarchy

---

# Approval Model (Critical)

## Core Principle

> Execution must always be governed, but approval should feel seamless and flexible.

---

## Approval Levels

### 1. Talents (Fully Trusted)

- Always allowed
- No approval required
- Pre-approved by definition

---

### 2. Methods (Conditionally Trusted)

- If approved once:
  - can be approved at task start
  - optional “auto-approve this method” toggle

- If not approved:
  - require approval on first execution

---

### 3. Unrecognized Actions

- Always require approval
- Must pass through Delegator

---

## Approval Surfaces (Flexible)

User can approve from anywhere:

- chat interface
- task manager
- notifications
- desktop (inline in EVOterminal / EVObrowser)
- mobile (quick approval prompts)

---

## Warp-Inspired Enhancement

Instead of:
> “Do you allow this command?”

We move to:

> “Here’s what I’m about to do. Approve?”

With:
- clear intent
- method context
- expected outcome

---

# Transparency Model

## Desktop Experience

User can:
- watch Alice operate in real-time
- see terminal actions
- see browser navigation
- view step-by-step execution

---

## Mobile Experience

User sees:
- summarized steps
- “play-by-play” updates
- progress states

Example:
- Logging in
- Navigating to reports
- Downloading file
- Saving to vault

---

## Optional Depth

User can choose:
- high-level summary
- detailed execution
- full transparency mode

---

# Tool Usage Philosophy

## Users should NOT learn tools

Users interact with:
- outcomes
- tasks
- approvals

NOT:
- terminal commands
- browser automation steps
- API calls

---

## Alice handles tools internally

Tools are:
- execution surfaces
- hidden behind Methods/Talents
- governed by Delegator

---

# Terminal Usage Rules

## EVOterminal

- Never first choice unless required
- Always governed by Delegator
- Requires:
  - method approval OR
  - explicit approval

---

## Safety Guardrails

- scoped execution only
- no unrestricted shell access
- full logging
- reversible actions where possible

---

# Browser Usage Rules

## EVObrowser

- Used when:
  - no API exists
  - API insufficient
  - plugin fails

- Must remain:
  - bounded
  - observable
  - auditable

---

# Vault Usage Rules

## EVOvault

- Never directly exposed
- Used only as:
  - capability provider
  - secure value injector

---

## Example Flow

User:
> “Download my monthly report”

Alice:

1. Detect plugin exists → try plugin
2. Plugin fails → fallback to browser
3. Needs login → request vault access approval
4. Execute browser steps
5. Save file
6. Return result

User sees:
- approval prompt
- progress updates
- final result

---

# Core Takeaway

> Plugin first. Browser if needed. Terminal when required. Manual as last resort.

With:

- learned preferences
- flexible approvals
- transparent execution
- governed safety

---

# The Real Outcome

This creates an experience where:

- beginners feel:
  > “It just works”

- advanced users feel:
  > “I can see and control everything”

- and the system remains:
  - safe
  - auditable
  - adaptive

#connect