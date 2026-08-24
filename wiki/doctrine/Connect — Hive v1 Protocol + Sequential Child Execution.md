---
title: Connect — Hive v1 Protocol + Sequential Child Execution
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Connect — Hive v1 Protocol + Sequential Child Execution.md
updated: 2026-07-24
---

# Connect — Hive v1 Protocol + Sequential Child Execution
Connect — Hive v1 + Runbook Execution
Parent MOC
[MOC - EVOconnect (Modular OS Layer)](https://www.notion.so/33ec72bad01381e0bcc2c52d91c18362)

Related Notes
[Connect Task System](https://www.notion.so/33dc72bad013813aad4feced36d43a98)
[Connect Hive Architecture](https://www.notion.so/33dc72bad013819996e4cee07376a3f2)
[Connect - Delegator & Governance](https://www.notion.so/33ec72bad013812bb1a2fcb216d082e3)
[Connect - Failure States & Resilience](https://www.notion.so/33ec72bad01381d6a053cffb914fcc09)
[Connect - UI Layer (Mobile)](https://www.notion.so/33ec72bad013816ca65bf53022a1e92d)
[Connect - UI Layer (Desktop)](https://www.notion.so/33ec72bad0138134b877c5f10bc07505)
[Connect - Control Panel & Tools](https://www.notion.so/33ec72bad0138179bbe9fcbf3f4e0994)
[EVOterminal - Core Design](https://www.notion.so/33ec72bad01381fd8f0dda89110a1a14)

Planned Notes (Not Created Yet)
These are future atomic notes that will be split out later:
Connect - Hive v1 (Primary Node Model)
Connect - Task Routing (Hive-lite)
Connect - Execution Engine
Connect - Runbook Execution
Connect - Live Execution UI
Connect - Logging System

Core Concept
Connect v1 enables cross-device execution using a primary node model,while introducing structured automation through Runbook Tasks.
This builds directly on:
[Connect Task System](https://www.notion.so/33dc72bad013813aad4feced36d43a98)
[Connect Hive Architecture](https://www.notion.so/33dc72bad013819996e4cee07376a3f2)
[Connect - Delegator & Governance](https://www.notion.so/33ec72bad013812bb1a2fcb216d082e3)

Hive v1 — Primary Node Model
Linked Concepts
[Connect Hive Architecture](https://www.notion.so/33dc72bad013819996e4cee07376a3f2)
[Connect - UI Layer (Mobile)](https://www.notion.so/33ec72bad013816ca65bf53022a1e92d)
[Connect - UI Layer (Desktop)](https://www.notion.so/33ec72bad0138134b877c5f10bc07505)

Core Idea
Phone = control surfaceComputer = execution nodeHive = coordination layer

Responsibilities
Hive handles:
Node awareness
Task routing
Execution coordination
Does NOT handle:
[[Connect Swarm Architecture]] (future)

Node Model
Primary Node
Desktop / Mac Mini
Runs:
[EVOterminal - Core Design](https://www.notion.so/33ec72bad01381fd8f0dda89110a1a14)
Internal browser
Executes all Alice Tasks

Client Nodes
Mobile ([[NOTION_PAGE:"[[Connect - UI Layer (Mobile)|Connect - UI Layer (Mobile)]]"]])
Desktop UI ([[NOTION_PAGE:"[[Connect - UI Layer (Desktop)|Connect - UI Layer (Desktop)]]"]])

Key Rule
One leader node at a time(ref: [[NOTION_PAGE:"[[Connect Hive Architecture|Connect Hive Architecture]]"]])

Task Routing (Hive-lite)
Linked Concepts
[Connect Task System](https://www.notion.so/33dc72bad013813aad4feced36d43a98)
[Connect Hive Architecture](https://www.notion.so/33dc72bad013819996e4cee07376a3f2)

Core Rule
All Alice Tasks → Primary Node

Task Lifecycle Extension
From: → [[NOTION_PAGE:"[[Connect Task System|Connect Task System]]"]]
Add: - routing - waiting_for_primary

Execution Engine
Linked Concepts
[EVOterminal - Core Design](https://www.notion.so/33ec72bad01381fd8f0dda89110a1a14)
[Connect - Delegator & Governance](https://www.notion.so/33ec72bad013812bb1a2fcb216d082e3)

Purpose
Execute approved Methods using:
Terminal
Browser

Principle
Execution is: - governed - method-bound - logged

Runbook Execution (Sequential Tasks)
Linked Concepts
[Connect Task System](https://www.notion.so/33dc72bad013813aad4feced36d43a98)
[Connect - Delegator & Governance](https://www.notion.so/33ec72bad013812bb1a2fcb216d082e3)

Core Idea
A parent task becomes an execution queue

Behavior
Ordered child tasks
One active child
Auto progression

Execution Flow
Parent → running
First child → in_progress
Child completes
Next child starts

Failure Handling
If a child fails:
Parent → paused
No further execution
(ref: [[NOTION_PAGE:"[[Connect - Failure States & Resilience|Connect - Failure States & Resilience]]"]])

Live Execution UI
Linked Concepts
[Connect - UI Layer (Mobile)](https://www.notion.so/33ec72bad013816ca65bf53022a1e92d)
[Connect - UI Layer (Desktop)](https://www.notion.so/33ec72bad0138134b877c5f10bc07505)

Purpose
Show execution in real time.

Elements
Node indicator (“Running on Mac Mini”)
Step progress
Log stream

Logging System
Linked Concepts
[Connect - Delegator & Governance](https://www.notion.so/33ec72bad013812bb1a2fcb216d082e3)
[Connect Task System](https://www.notion.so/33dc72bad013813aad4feced36d43a98)

Purpose
Ensure:
auditability
traceability
transparency

Relationship Summary
Builds On
[Connect Task System](https://www.notion.so/33dc72bad013813aad4feced36d43a98)
[Connect Hive Architecture](https://www.notion.so/33dc72bad013819996e4cee07376a3f2)
[Connect - Delegator & Governance](https://www.notion.so/33ec72bad013812bb1a2fcb216d082e3)

Extends
[Connect - UI Layer (Mobile)](https://www.notion.so/33ec72bad013816ca65bf53022a1e92d)
[Connect - UI Layer (Desktop)](https://www.notion.so/33ec72bad0138134b877c5f10bc07505)

Enables
Runbook Execution (this note)
Future: [[Connect Swarm Architecture]]

Core Insight
Hive coordinatesExecution Engine actsRunbooks structure
Together:
Tasks become executable systems

## Related

^[source-materials/mirrors/doctrine/Connect — Hive v1 Protocol + Sequential Child Execution.md]
