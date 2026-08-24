---
title: EVOconnect — Placement of EVOterminal, EVObrowser, and EVOvault
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/EVOconnect — Placement of EVOterminal, EVObrowser, and EVOvault.md"]
updated: 2026-07-24
---

# EVOconnect — Placement of EVOterminal, EVObrowser, and EVOvault
Core Question
Where do EVOterminal, EVObrowser, and EVOvault fit in the Connect architecture?
They are not separate apps in the core model.
They are:
bounded execution and access surfaces inside EVOconnect
This fits the current Connect architecture, where Connect is the modular OS layer that links Alice to apps, tools, devices, and compute in a governed, privacy-first way oai_citation:0‡MOC - EVOconnect (Modular OS Layer).md

Short Answer
EVOterminal
A governed local execution surface for environment access.
EVObrowser
A governed web interaction surface for human-facing websites and browser workflows.
EVOvault
A protected secrets and sensitive-data layer that supplies capabilities and values without exposing raw secrets to Alice.
Together, they sit under:
Connect control layer + Delegator governance + task execution
They are how Alice acts in the world without becoming unrestricted.

High-Level Placement
1. Connect is the orchestrator
Connect already owns: - task delegation - bounded automation - Hive orchestration - Swarm acceleration - control-panel style interaction oai_citation:1‡MOC - EVOconnect (Modular OS Layer).md
So these three belong inside the Connect execution model, not outside it.

2. Delegator sits above all three
Delegator is the safety layer.
It already defines that Alice cannot: - access unrestricted terminal - modify system files - execute arbitrary scripts - access external data without approval oai_citation:2‡Connect - Delegator & Governance.md
That means:
EVOterminal
must be a Delegator-governed terminal surface
EVObrowser
must be a Delegator-governed browser surface
EVOvault
must be a Delegator-governed secrets/capabilities surface
None of them should bypass Delegator.

3. They are execution surfaces, not user modes
This is important.
You already defined that Connect should avoid fragmented mode-based thinking and instead present one Alice across multiple surfaces oai_citation:3‡EVOconnect — Continuous Consciousness vs Mode-Based Systems.md
So these are not: - “terminal mode” - “browser mode” - “vault mode”
They are:
surfaces Alice can use when the task requires them
The user should still feel like: - they are talking to Alice - Alice is handling the task - Connect is coordinating the work
Not: - “now I have to switch into a technical tool”

Role of Each Component
EVOterminal
Purpose
Provide a governed, sandboxed local execution surface.
Your current note already frames it correctly: - internal terminal interface - sandboxed execution environment - governed by Delegator - fully logged - scoped to approved actions oai_citation:4‡EVOterminal - Core Design.md
Architectural Role
EVOterminal is the environment access layer for: - local repositories - local scripts - shell-like workflows - desktop tooling that cannot be handled through provider APIs
Best placement
Inside Connect’s bounded tools layer.
That lines up with Connect’s current tools model: - internal browser - internal terminal sandbox - tool registry - plugin system - all governed, logged, and scoped oai_citation:5‡Connect - Control Panel & Tools.md
Practical interpretation
EVOterminal should be treated as: - a controlled execution backend - a tool host for Methods/Talents - a bridge to local systems when provider/API access is unavailable
Not: - a user-facing developer terminal - a raw shell - a separate power-user app

EVObrowser
Purpose
Provide a governed web execution surface.
This is the equivalent of: - browser-based workflows - website navigation - human-facing SaaS interaction - web steps that have no API or no useful API path
Architectural Role
EVObrowser should be the web counterpart to EVOterminal.
If EVOterminal handles: - local environment workflows
Then EVObrowser handles: - browser/UI workflows
Best placement
Also inside Connect’s bounded tools layer, alongside EVOterminal.
This matches your broader “provider vs environment” split: - provider access when OAuth/API/local model exists - environment access when only human-facing interfaces exist
Practical interpretation
EVObrowser is how Alice can: - use a site the way a person would - complete steps visually - operate through governed browser actions
Not: - a full unrestricted browser - a general browsing app for the user - a bypass around plugins/providers

EVOvault
Purpose
Provide secure handling of: - secrets - tokens - personal information - protected values - sensitive credentials
Architectural Role
EVOvault is not really an execution surface in the same sense as terminal/browser.
It is a:
protected capability and secret source
It should sit between: - plugins - tasks - methods/talents - provider/environment actions
and the raw sensitive data they may require.
Best placement
Under Connect’s security/governance layer, adjacent to Delegator, and consumed by tools/plugins when approved.
This fits Connect’s privacy model: - local-first processing - no cross-user data visibility - no telemetry harvesting - user ownership of data and control oai_citation:6‡Connect - Security & Privacy Model.md
Practical interpretation
Alice should not “know passwords.”
Instead: - a Method requests a capability - Delegator validates it - EVOvault supplies the needed secret/value in a bounded way - the action executes - the secret is never casually exposed into chat/runtime context
So EVOvault is closer to: - secure capability broker - protected value provider - governed identity/secrets layer
Not: - a notes app - a raw password manager UI replacement - a general storage drawer Alice can freely rummage through

The Layer Model
Layer 1 — User Intent
User asks for an outcome: - “Send this invoice” - “Update that spreadsheet” - “Log into the vendor portal and download the report”

Layer 2 — Task System
Connect turns that into: - My Task or Alice Task - lifecycle tracking - approval checkpoints oai_citation:7‡Connect - Task System.md

Layer 3 — Delegator
Delegator decides: - what is allowed - whether approval is required - which surface can be used - whether secrets or protected access may be used oai_citation:8‡Connect - Delegator & Governance.md

Layer 4 — Execution Surface Selection
If structured provider access exists
Use plugin/provider integration
If local environment access is needed
Use EVOterminal
If human-facing web flow is needed
Use EVObrowser
If protected values are needed
Use EVOvault
This aligns directly with your provider/environment distinction: - use provider access first when reliable - environment access when necessary - manual fallback last oai_citation:9‡Provider vs Environment Access.md

Layer 5 — Method / Talent Execution
Alice executes the approved Method: - with tools - with browser/terminal steps - with vault-backed values if allowed - fully logged and auditable

Relationship to Plugins
This is where it gets important.
Plugins
Plugins are how Alice gains bounded access to outside systems through: - connection - capabilities - resources oai_citation:10‡EVOconnect — Plugin Model (Capabilities, Connectors, and User File Access).md
EVOterminal / EVObrowser
These are not plugins themselves in the normal sense.
They are more like:
core environment adapters / internal execution surfaces
A plugin might say: - “QuickBooks API available” - “Google Drive available”
But if no useful provider exists, Alice may need: - EVObrowser for web interaction - EVOterminal for local automation
So:
Plugins
connect Alice to external structured systems
EVOterminal / EVObrowser
let Alice operate in unstructured or human-facing environments
EVOvault
provides secure protected values to either path

Relationship to Talents and Methods
These three should almost never appear to the user as raw tools.
Instead, they should show up as: - part of a Method - part of a Talent - part of task execution
That fits your tool abstraction principle:
users should not need to learn the tools; Alice should use them behind the scenes oai_citation:11‡EVOconnect — Tool Abstraction & Outcome-Oriented Computing.md
So the user should see: - proposed method - permissions requested - expected outcome
Not: - terminal commands - browser automation internals - secret handling details
Unless they explicitly want advanced visibility.

Relationship to Headless Usability
You’ve said Connect should be so simple that even someone who has never used a computer could still use it.
That means:
EVOterminal
must never require terminal knowledge from the user
EVObrowser
must never require “now go click around this site yourself” as the primary model
EVOvault
must never make the user reason like a sysadmin about credentials and secret injection
These systems exist to preserve power without leaking complexity.

Recommended Final Placement
EVOterminal
Category: Internal bounded tool / environment surfaceOwned by: Connect tools layerGoverned by: DelegatorUsed by: Methods, Talents, Alice TasksPurpose: local controlled execution
EVObrowser
Category: Internal bounded tool / web execution surfaceOwned by: Connect tools layerGoverned by: DelegatorUsed by: Methods, Talents, Alice TasksPurpose: human-facing web workflows
EVOvault
Category: Security and capability infrastructureOwned by: Connect security/governance layerGoverned by: Delegator + approval rulesUsed by: plugins, Methods, Talents, environment/provider actionsPurpose: protected access to secrets and sensitive values

The Best Mental Model
If Connect is the modular OS layer and Alice is the orchestrator:
EVOterminal
is her governed hand for local systems
EVObrowser
is her governed hand for the web
EVOvault
is her governed access to protected identity and secrets
Together, they let her act powerfully without becoming unsafe.

Core Takeaway
EVOterminal and EVObrowser are bounded execution surfaces inside Connect.EVOvault is the protected secret/capability layer that supports them.All three sit under Delegator and task execution, not outside the architecture.
They are not separate modes.
They are: - governed surfaces - selected by Method - constrained by Delegator - hidden behind outcome-oriented interaction
That keeps Connect: - simple to use - powerful in practice - and aligned with your privacy-first architecture.
#connect

## Related
