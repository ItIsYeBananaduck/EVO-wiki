# Connect — Hive v1 + Runbook Execution

## Parent MOC
- [[MOC - EVOconnect (Modular OS Layer)]]

---

## Related Notes
- [[Connect - Task System]]
- [[Connect - Hive Architecture]]
- [[Connect - Delegator & Governance]]
- [[Connect - Failure States & Resilience]]
- [[Connect - UI Layer (Mobile)]]
- [[Connect - UI Layer (Desktop)]]
- [[Connect - Control Panel & Tools]]
- [[EVOterminal - Core Design]]

---

## Planned Notes (Not Created Yet)

These are **future atomic notes** that will be split out later:

- Connect - Hive v1 (Primary Node Model)
- Connect - Task Routing (Hive-lite)
- Connect - Execution Engine
- Connect - Runbook Execution
- Connect - Live Execution UI
- Connect - Logging System

---

# Core Concept

> Connect v1 enables cross-device execution using a primary node model,  
while introducing structured automation through Runbook Tasks.

This builds directly on:

- [[Connect - Task System]]
- [[Connect - Hive Architecture]]
- [[Connect - Delegator & Governance]]

---

# Hive v1 — Primary Node Model

## Linked Concepts
- [[Connect - Hive Architecture]]
- [[Connect - UI Layer (Mobile)]]
- [[Connect - UI Layer (Desktop)]]

---

## Core Idea

> Phone = control surface  
> Computer = execution node  
> Hive = coordination layer  

---

## Responsibilities

Hive handles:

- Node awareness
- Task routing
- Execution coordination

Does NOT handle:

- [[Connect - Swarm Architecture]] (future)

---

## Node Model

### Primary Node
- Desktop / Mac Mini
- Runs:
  - [[EVOterminal - Core Design]]
  - Internal browser
- Executes all Alice Tasks

---

### Client Nodes
- Mobile ([[Connect - UI Layer (Mobile)]])
- Desktop UI ([[Connect - UI Layer (Desktop)]])

---

## Key Rule

> One leader node at a time  
(ref: [[Connect - Hive Architecture]])

---

# Task Routing (Hive-lite)

## Linked Concepts
- [[Connect - Task System]]
- [[Connect - Hive Architecture]]

---

## Core Rule

> All Alice Tasks → Primary Node

---

## Task Lifecycle Extension

From:
→ [[Connect - Task System]]

Add:
- routing
- waiting_for_primary

---

# Execution Engine

## Linked Concepts
- [[EVOterminal - Core Design]]
- [[Connect - Delegator & Governance]]

---

## Purpose

Execute approved Methods using:

- Terminal
- Browser

---

## Principle

> Execution is:
- governed
- method-bound
- logged

---

# Runbook Execution (Sequential Tasks)

## Linked Concepts
- [[Connect - Task System]]
- [[Connect - Delegator & Governance]]

---

## Core Idea

> A parent task becomes an execution queue

---

## Behavior

- Ordered child tasks
- One active child
- Auto progression

---

## Execution Flow

1. Parent → running  
2. First child → in_progress  
3. Child completes  
4. Next child starts  

---

## Failure Handling

If a child fails:

- Parent → paused
- No further execution

(ref: [[Connect - Failure States & Resilience]])

---

# Live Execution UI

## Linked Concepts
- [[Connect - UI Layer (Mobile)]]
- [[Connect - UI Layer (Desktop)]]

---

## Purpose

Show execution in real time.

---

## Elements

- Node indicator (“Running on Mac Mini”)
- Step progress
- Log stream

---

# Logging System

## Linked Concepts
- [[Connect - Delegator & Governance]]
- [[Connect - Task System]]

---

## Purpose

Ensure:

- auditability
- traceability
- transparency

---

# Relationship Summary

## Builds On
- [[Connect - Task System]]
- [[Connect - Hive Architecture]]
- [[Connect - Delegator & Governance]]

---

## Extends
- [[Connect - UI Layer (Mobile)]]
- [[Connect - UI Layer (Desktop)]]

---

## Enables
- Runbook Execution (this note)
- Future: [[Connect - Swarm Architecture]]

---

# Core Insight

> Hive coordinates  
> Execution Engine acts  
> Runbooks structure  

Together:

> **Tasks become executable systems**