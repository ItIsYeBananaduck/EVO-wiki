---
title: "Evoconnect — Workflow Automation Through Specialist Orchestration And Internalization"
type: doctrine
tags: ['lsctech', 'doctrine', 'source-material', 'evo']
updated: 2026-08-24
---

#connect
## Core Idea

> The current multi-agent workflow is good, but too tedious.
> EVOconnect should automate that workflow through Alice so the user no longer has to manually hand work from tool to tool.

The long-term goal is not just orchestration.

The long-term goal is:

> Alice learns the workflow, runs it, and gradually internalizes it so she depends less on outside specialists over time.

---

## Problem

The current workflow works, but it is heavy.

It requires repeated manual handoff:

- architecture is defined in one place
- implementation is sent to another agent
- review is handled by another tool
- fixes are checked again
- planning is passed to Claude
- issues are passed to Linear
- code is reviewed by CodeRabbit
- the user acts as the router between all of them

This creates friction even when the workflow itself is sound.

The problem is not that handoff is bad.

The problem is:

> the user is still doing too much of the orchestration manually.

---

## Vision

> Connect should make the workflow feel seamless.

Instead of the user acting like a human router, Alice should:

- understand the workflow
- know which specialist handles which step
- send the right artifact to the right agent
- wait for the next event
- interpret the result
- continue the workflow automatically
- ask for approval only when necessary
- learn from the specialists while doing all of this

---

## Core Principle

> A workflow should only need to be defined once.

After that, Alice should be able to:

- follow it repeatedly
- adapt it to the user
- improve her execution over time
- internalize parts of it as confidence grows

This is one of the clearest examples of what Connect is supposed to become.

---

## The Current Workflow as Training Data

The current workflow is not just a temporary setup.

It is also:

> the training template for Connect.

Example workflow:

1. create the issue
2. assign the issue
3. wait for CodeRabbit review
4. fix the issues CodeRabbit found
5. commit and push or prepare merge
6. hand the larger planning packet to Claude
7. get back implementation guidance
8. convert that into structured issues in Linear
9. continue the loop until complete

Today, the user performs that routing.

Later, Alice should perform it.

---

## Why This Matters

### 1. It removes orchestration burden from the user

The user should not have to constantly think:

- who gets this task
- when to hand it off
- what context to send
- when to wait
- when to escalate
- when to switch tools

Alice should own that.

---

### 2. It lets specialists train Alice

The important idea is not just that Alice uses specialists.

It is that:

> every interaction with a specialist becomes a learning opportunity.

When Alice hands work to:
- Codex
- Claude
- CodeRabbit
- Linear agent
- any future specialist

she should observe:
- what they did
- what pattern they followed
- why their result was accepted or rejected
- what inputs produced good outputs
- what errors required escalation

Over time, that lets her reproduce more of the workflow herself.

---

### 3. It creates user-specific specialization

This is not about making one universal coding specialist.

This is about making:

> a specialist for the individual user.

If one user has a coding workflow, Alice learns that.

If another user has a business workflow, Alice learns that.

If another user has a content workflow, Alice learns that.

That means Connect is not just agent orchestration.

It is:

> personalized workflow internalization.

---

## The Long-Term Behavior

Alice should progress through stages.

### Stage 1 — Manual orchestration by the user
The user decides:
- which agent to use
- when to escalate
- when to move to the next step

### Stage 2 — Alice-assisted orchestration
Alice suggests:
- which agent should handle the next step
- what packet to send
- what the workflow state is
- what approval is needed

### Stage 3 — Alice-run orchestration
Alice:
- runs the workflow
- routes work automatically
- waits for responses
- handles review loops
- surfaces approvals only when needed

### Stage 4 — Internalized execution
Alice can perform more of the workflow herself because she has learned:
- the order of steps
- common patterns
- common fixes
- common review outcomes
- how the user prefers the work to be done

At that point, specialists become:
- fallback support
- escalation targets
- edge-case helpers

not the default path for every step.

---

## Workflow Definition Model

A workflow should be represented as a structured sequence of steps.

Each step should include:

- step name
- purpose
- required inputs
- actor or specialist
- completion signal
- next step
- failure path
- escalation path
- learning value

Example conceptually:

- create issue
- assign issue
- wait for review
- interpret review
- apply fix
- re-run checks
- escalate if needed
- prepare merge
- update task state
- close loop

This gives Alice a defined path she can follow.

---

## What Alice Should Learn at Each Step

At every step, Alice should capture:

### Inputs
- what information was sent
- what context was required
- what was unnecessary

### Outputs
- what came back
- whether it was accepted
- whether it caused new issues

### Patterns
- common review findings
- common implementation structures
- common escalation triggers
- common sequencing decisions

### User preferences
- whether the user prefers one agent over another
- what level of confidence is required
- when to ask vs when to proceed
- when to stop and surface the task

This is how the workflow becomes personalized.

---

## Relationship to Connect

This is one of the clearest expressions of Connect’s role.

Connect is not just:
- chat
- plugins
- tools
- tasks
- Hive orchestration

It is also:

> the system that allows Alice to observe, run, and learn workflows across specialists until she can perform more of them herself.

That means Connect must support:

- workflow definitions
- step execution
- specialist routing
- waiting states
- result interpretation
- approval gates
- learning capture
- internalization thresholds

---

## Relationship to Methods and Talents

This workflow learning model fits naturally into Methods and Talents.

### Method
A defined way of doing something that can be followed and reviewed.

### Talent
A method that has been validated enough to become trusted and reusable.

That means a workflow can begin as:
- a user-defined process
- a repeated orchestration pattern

Then over time:
- become a Method
- then become a Talent
- then become partially or fully internalized by Alice

This gives workflow learning a natural progression inside the Connect architecture.

---

## Key Architectural Principle

> External specialists are not just helpers.
> They are temporary teachers.

Alice should use them to:
- extend capability
- complete work safely
- learn patterns
- reduce future dependence

This is one of the most important parts of the Connect vision.

---

## User-Specific Workflow Intelligence

The system should be able to support different workflow types for different users.

Examples:

### Coding user
- issue creation
- implementation routing
- review loop
- fix loop
- planning packet generation
- merge preparation

### Business user
- task creation
- document generation
- approval routing
- accounting tool interaction
- scheduling and follow-up

### Operations user
- checklist execution
- status tracking
- external system actions
- exception handling
- reporting

The structure is the same.

Only the workflow content changes.

That is why this becomes so powerful.

---

## Core Takeaway

> The current workflow is tedious because the user is still the orchestrator.

Connect should change that.

Alice should:
- run the workflow
- route to specialists
- learn from specialists
- personalize the flow
- internalize repeated patterns
- reduce dependence over time

This is not just automation.

It is:

> personalized workflow learning through governed orchestration.

And that is one of the biggest reasons Connect can become transformational.