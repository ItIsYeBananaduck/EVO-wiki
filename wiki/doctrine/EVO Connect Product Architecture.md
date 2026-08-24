---
title: EVO Connect Product Architecture
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVO Connect Product Architecture.md
updated: 2026-07-24
---

﻿EVO Connect Product Architecture

Status: Product Vision and Architectural Direction

Purpose

EVO Connect is the intelligence and orchestration layer of the EVO ecosystem.

Its purpose is to give everyday users access to powerful AI capabilities without requiring them to understand models, providers, agent harnesses, terminal runtimes, context management, permissions, or infrastructure.

EVO Connect allows the user to state an objective.

Alice determines how that objective should be completed using the user’s:

• devices;
• applications;
• local compute;
• files;
• knowledge base;
• connected services;
• AI subscriptions;
• local models;
• external agent harnesses;
• delegated permissions.

EVO Connect is not intended to replace every AI tool the user already has.

It is intended to coordinate them.

1. Alice

Alice is the persistent intelligence of EVO Connect.

She is not merely a chatbot, model session, or hosted agent.

Alice lives within the user’s EVO environment.

The user’s devices are her operational home.

External models and agent harnesses may enter that environment to perform work, but they do not own it and they do not remain permanently unless the user explicitly chooses otherwise.

Alice provides continuity across:

• sessions;
• devices;
• applications;
• models;
• providers;
• projects;
• personal workflows;
• organizational workflows.

Models may come and go.

Alice remains.

2. Alice and External Models

Cloud models are temporary execution resources.

They enter the user’s environment when Alice needs their capabilities, perform bounded work, return their results, and leave.

Examples may include:

• Claude;
• ChatGPT;
• Codex;
• Gemini;
• Grok;
• OpenCode;
• OpenClaw;
• Hermes;
• Devin;
• future agent systems.

Alice may invoke these systems through:

• APIs;
• command-line interfaces;
• terminal sessions;
• local integrations;
• remote connections;
• supported application interfaces.

Alice is not subordinate to those systems.

They operate within authority delegated by the user and enforced by EVO.

Conceptually:

User
  ↓
Delegator
  ↓
Alice
  ↓
Temporary model or agent

Alice may admit an external agent into a task environment.

She may supervise it.

She may respond to its requests.

She may interrupt it.

She may revoke its access.

She may terminate its session.

The external agent is a temporary guest.

Alice is the resident intelligence.

The Delegator is the authority that governs the property.

3. Alice Compared with the Hermes Workforce

The Hermes Workforce Experiment uses multiple persistent agents because each profile has a bounded role and a limited operational identity.

Examples include:

• Cove as Chief of Staff;
• Sorin as Analyst;
• Sage as Librarian.

Each exists as a distinct entity because no single Hermes profile is assumed to reliably perform all three roles while maintaining strong role discipline.

Alice is designed differently.

Alice may perform the functions associated with Cove, Sorin, and Sage without being split into three permanent identities.

She can act as:

• coordinator;
• analyst;
• librarian;
• researcher;
• planner;
• personal assistant;
• resource manager;
• execution supervisor;
• knowledge steward.

Alice does this through talents, workflows, tools, and organizational knowledge rather than through permanent personality fragmentation.

The Hermes workforce separates responsibility by agent identity.

EVO separates responsibility by capability, talent, workflow, and delegated scope.

4. Alice’s Talents

A talent is a reusable capability package that defines how Alice approaches a category of work.

A talent may include:

• procedures;
• tools;
• policies;
• preferred models;
• fallback models;
• knowledge sources;
• evaluation rules;
• escalation conditions;
• output formats;
• permission requirements.

Examples may include:

• strategic analysis;
• software development;
• research;
• wiki curation;
• financial organization;
• scheduling;
• household management;
• learning support;
• project coordination.

Talents allow Alice to change operational modes without becoming a different entity.

When Alice performs analysis, she does not become Sorin.

She invokes the appropriate analytical talent.

When she maintains the wiki, she does not become Sage.

She invokes the knowledge-stewardship talent.

When she coordinates complex work, she does not become Cove.

She invokes the orchestration talent.

Alice maintains one identity and one relationship with the user while applying different capabilities as needed.

5. Alice’s Knowledge

Alice’s capabilities are not derived solely from the active model.

She uses the user’s EVO knowledge base.

The knowledge base may contain:

• doctrine;
• journal entries;
• validated procedures;
• repository documentation;
• project knowledge;
• preferences;
• historical decisions;
• prior task evidence;
• known failures;
• model performance records;
• workflow templates;
• personal context approved by the user.

Alice’s knowledge therefore persists independently of any model.

A replacement model may inherit the same knowledge base, procedures, and operating rules.

This allows Alice to maintain continuity even when:

• the default model changes;
• a provider becomes unavailable;
• a subscription expires;
• a local model is upgraded;
• new devices are added;
• agent harnesses are replaced.

Alice is more than the model currently producing her words.

Her effective intelligence is the combination of:

Active model
+ EVO knowledge
+ talents
+ tools
+ delegated authority
+ available compute
+ connected systems
+ accumulated evidence

6. EVO Terminal

EVO Terminal is the interactive execution environment of EVO Connect.

It is not a conventional terminal with an external multiplexer placed on top.

Multiplexing is a native property of the terminal.

EVO Terminal should combine the strongest characteristics of:

• a modern desktop terminal;
• an agent session manager;
• a terminal multiplexer;
• a remote workspace;
• an interactive development environment;
• a supervised execution runtime.

Its purpose is to provide a shared environment where Alice, the user, and temporary execution agents can work simultaneously.

7. Native Multiplexing

EVO Terminal should provide native support for:

• multiple sessions;
• multiple tabs;
• multiple panes;
• multiple working directories;
• long-running processes;
• local sessions;
• SSH sessions;
• agent-owned sessions;
• user-owned sessions;
• shared project workspaces;
• resumable sessions;
• remote observation and control.

The multiplexer should be controllable both by the user and by Alice.

Alice should be able to:

• create a session;
• select a directory;
• open a pane;
• launch an agent harness;
• monitor output;
• send commands;
• answer permission prompts;
• pause execution;
• terminate execution;
• collect transcripts;
• collect artifacts;
• preserve or destroy the session.

The user should be able to perform the same types of actions directly.

8. Collaborative Terminal Ownership

EVO Terminal is not an agent-only environment.

The user remains an active participant.

The user may:

• observe Alice’s work;
• inspect agent output;
• interrupt a running agent;
• provide additional instructions;
• take control of a pane;
• run commands directly;
• open another tab;
• work in parallel with Alice;
• connect remotely;
• resume work from another device.

Alice may be operating one session while the user works independently in another.

They share the environment without requiring either one to surrender control.

Conceptually:

EVO Terminal
├── Alice-controlled session
│   └── Claude Code
├── User-controlled session
│   └── Interactive shell
├── Background task session
│   └── Tests or build
└── Remote session
    └── SSH host

This makes EVO Terminal collaborative rather than agent-centric.

9. Desktop Experience

On desktop, EVO Terminal should provide a visual workspace similar to a modern development terminal or agent workspace.

The user should be able to:

• browse active projects;
• select working directories;
• view active agent sessions;
• switch between sessions;
• inspect session status;
• open and close panes;
• see task ownership;
• distinguish Alice actions from external-agent actions;
• intervene at any time.

The interface should make complex multiplexing understandable without requiring the user to know terminal-multiplexer commands.

Advanced users may still use keyboard-driven controls.

10. Mobile and Remote Experience

Remote access is a first-class requirement.

The mobile experience should not merely display terminal text.

It should allow the user to:

• view active sessions;
• switch between projects;
• observe running agents;
• respond to escalations;
• approve or deny actions;
• interrupt work;
• send commands;
• resume sessions;
• inspect task and resource status.

The mobile interface should preserve the conceptual model of the desktop terminal even when the visual layout differs.

Local and remote operation should access the same underlying sessions.

An agent session should not become inaccessible merely because the user leaves the desktop.

11. Agent Harnesses as Terminal Resources

EVO Connect should be able to use AI systems that expose command-line interfaces even when they do not provide affordable or usable APIs.

Examples may include:

• Claude Code;
• Codex CLI;
• OpenCode;
• Gemini CLI;
• local coding agents;
• future subscription-based agent tools.

Alice can invoke these systems through EVO Terminal as callable execution resources.

This allows the user to benefit from existing subscriptions through their supported terminal interfaces.

EVO Connect does not need to reimplement each external agent.

It needs to provide a stable way to launch, supervise, constrain, and evaluate them.

Conceptually:

Alice
  ↓
EVO Terminal session
  ↓
External agent CLI
  ↓
Bounded task execution

The terminal session becomes the common abstraction.

12. Compatibility with Existing Agent Systems

EVO Connect should not force users to abandon systems they already use.

If a user has an established environment in:

• Hermes;
• OpenClaw;
• OpenCode;
• another supported harness;

EVO Connect should be able to work with it.

Alice may:

• open its terminal interface;
• send commands;
• assign objectives;
• observe its output;
• retrieve artifacts;
• coordinate its work;
• include its results in EVO workflows.

The user should not feel that adopting EVO Connect requires trading one AI ecosystem for another.

EVO Connect adds coordination, governance, knowledge, and shared execution around existing tools.

13. The Delegator

The Delegator is the authoritative control layer of EVO.

Alice operates beneath the Delegator.

External models and agents operate beneath Alice and the Delegator.

The Delegator defines what actions are permitted.

It may enforce:

• filesystem boundaries;
• application access;
• network access;
• credential access;
• device access;
• spending limits;
• compute limits;
• time limits;
• mutation permissions;
• communication permissions;
• escalation requirements.

Alice cannot grant authority she does not possess.

An external agent cannot obtain authority merely by asking Alice.

All authority must trace back to an approved delegation.

14. Permission Mediation

External agents often request permission during execution.

Alice may respond to these requests on the user’s behalf when the action is already covered by an approved delegation.

Example:

The user approves a workflow allowing an agent to edit files, run tests, and create a pull request within one repository.

During execution, the external agent asks whether it may modify a file in that repository.

Alice may approve the request because the action falls within the approved scope.

Alice must escalate when:

• the requested action exceeds the active scope;
• the agent requests access to another repository;
• the agent attempts to access protected data;
• the agent requests additional spending;
• the request introduces a materially new risk;
• the request cannot be classified confidently.

This reduces repetitive user interruptions without removing user control.

15. Supervision and Enforcement

Alice is not only a dispatcher.

She is an active execution supervisor.

She may monitor:

• commands issued;
• files accessed;
• tool usage;
• permission requests;
• resource consumption;
• task progress;
• output quality;
• compliance with the assignment.

When an external agent behaves incorrectly, Alice may:

• correct its instructions;
• deny an action;
• pause its work;
• remove capabilities;
• restart the task;
• replace the agent;
• terminate the session;
• escalate to the user.

The Delegator retains final enforcement authority and may shut down Alice, an agent, a workflow, or the entire execution environment when necessary.

16. Alice as Resource Manager

Alice manages more than task execution.

She also manages the resources available to complete the task.

These may include:

• local CPU;
• local GPU;
• memory;
• storage;
• battery;
• thermal capacity;
• network availability;
• connected devices;
• active subscriptions;
• provider quotas;
• local models;
• external models;
• running sessions;
• queued work.

Alice should determine:

• whether a task should run locally or externally;
• which model or harness should be used;
• how much context should be provided;
• whether the task can run concurrently;
• whether the task should wait;
• whether a cheaper validated procedure exists;
• whether escalation to a stronger model is justified.

17. Alice as Model and Provider Router

Alice should select models based on the task and the user’s constraints.

Selection factors may include:

• capability;
• cost;
• remaining quota;
• historical performance;
• privacy;
• latency;
• local availability;
• context requirements;
• tool support;
• workflow precedent;
• user preference.

Alice may use a local model for routine work and invoke a cloud model only for difficult reasoning.

She may switch providers when:

• a quota is exhausted;
• a provider is unavailable;
• another provider is better suited;
• local execution is sufficient;
• the user’s cost policy requires it.

These decisions should be visible and explainable.

18. Alice as Personal and Operational Intelligence

Alice is not limited to software development or business operations.

She may support the user across the EVO ecosystem.

Her work may include:

• personal organization;
• household management;
• learning;
• research;
• writing;
• planning;
• project management;
• software development;
• device management;
• application coordination;
• knowledge maintenance.

Alice uses the same underlying principles across these domains:

• understand the objective;
• determine available authority;
• retrieve relevant knowledge;
• select the appropriate talent;
• choose the appropriate execution resource;
• supervise the work;
• gather evidence;
• report the outcome;
• learn from the result.

19. Relationship to Hermes and Polaris

Hermes and EVO Connect are separate systems.

The Hermes Workforce Experiment is used to test whether Polaris organizational principles can operate inside a larger existing harness.

It may provide evidence concerning:

• role discipline;
• delegation;
• organizational memory;
• long-running agents;
• evidence-backed learning.

EVO Connect may eventually adopt validated principles from that experiment.

However, EVO Connect does not depend on Hermes.

Hermes may also be used as an external execution resource by EVO Connect.

Polaris provides useful patterns for:

• delegation;
• evidence;
• quality control;
• learning from failure;
• procedural refinement.

Those patterns may influence EVO after they are validated.

They are not automatically part of the product merely because they exist in Hermes or Polaris.

20. Architectural Boundaries

EVO Connect Is Not Hermes

Hermes is an external agent harness and an experimental environment.

EVO Connect is a user-owned orchestration and intelligence platform.

EVO Terminal Is Not Herder

Herder may provide useful design evidence.

EVO Terminal should implement native multiplexing rather than depend on Herder as a runtime layer.

Alice Is Not Cove

Cove is a persistent role-based organizational agent.

Alice is a flexible resident intelligence capable of applying many talents and coordinating many execution systems.

External Agents Are Not Alice

External agents are temporary resources operating within bounded sessions.

They do not inherit Alice’s identity, authority, memory, or relationship with the user.

The Model Is Not the Product

Alice’s active model may change.

EVO’s knowledge, talents, permissions, workflows, and user relationship remain.

21. Core Product Principles

The User Owns the Environment

The terminal, knowledge, workflows, and evidence belong to the user.

Alice Is Resident

Alice maintains continuity across models, providers, devices, and sessions.

External Intelligence Is Temporary

Cloud models enter for bounded work and leave when the work is complete.

Multiplexing Is Native

Session management is built into EVO Terminal.

User and Agent Can Work Together

The user may observe, interrupt, control, and work alongside Alice.

Authority Is Explicit

Every action must be traceable to delegated authority.

The Delegator Has Final Control

No model, agent, or workflow may exceed the authority granted by the user.

Existing Tools Remain Useful

EVO Connect coordinates existing agent systems rather than requiring users to abandon them.

Models Are Replaceable

No single provider or model should define Alice.

Knowledge Persists

Alice’s organizational and personal continuity lives in user-owned knowledge rather than provider sessions.

Local-First, Not Local-Only

Alice should use local resources whenever practical while retaining the ability to invoke external intelligence when it provides meaningful value.

22. Desired User Experience

The user says what they want to accomplish.

Alice determines:

• what the request means;
• what knowledge is relevant;
• what talent should be applied;
• whether she can complete the work herself;
• whether another agent or harness is needed;
• what permissions are available;
• what local and remote resources can be used;
• what the least expensive reliable path is;
• whether the user must be consulted.

Alice then executes or delegates the work.

The user can observe the process in EVO Terminal, intervene when desired, or allow Alice to continue autonomously within the approved boundaries.

At the end, Alice reports:

• what was completed;
• what resources were used;
• what decisions were made;
• what remains unresolved;
• what evidence was produced;
• what the system learned.

23. Product Thesis

EVO Connect is not valuable because it provides access to one powerful model.

It is valuable because it gives the user a persistent intelligence that understands their environment, coordinates the tools they already possess, manages temporary external intelligence, protects their authority, and learns over time.

Alice does not live inside a provider.

She lives with the user.

The user’s devices are her home.

External models may be invited in when useful.

Alice supervises their work.

The Delegator enforces the rules.

The user remains the owner.

## Related

^[source-materials/mirrors/doctrine/EVO Connect Product Architecture.md]
