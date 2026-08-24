---
title: Connect - Delegator & Governance
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Connect - Delegator & Governance.md
updated: 2026-07-24
---

# Connect - Delegator & Governance
Core Principle
Delegator is the safety layer of EVOconnect.
Delegator ensures that all actions taken by Alice are:
bounded
auditable
intentional
reversible when possible

🔒 Hard Constraints
Alice cannot:
access an unrestricted terminal
modify system-critical files
execute arbitrary scripts
access external data without approval
perform actions outside defined Talents

Law
If an action is not defined, it is not allowed.

⚙️ Execution Rules
All actions must:
bind to a Method
execute through an approved Talent
follow a defined scope
be auditable

Clarification
Method = how the action is performed
Talent = what capability is being used
Delegator = enforces both

Law
No Method, no execution.

🧩 Governance Responsibilities
Delegator ensures:

1. No Rogue Behavior
Alice cannot improvise outside defined Methods
All actions must be pre-structured

2. No Silent Execution
actions must be visible or logged
user-impacting actions must be traceable

3. No Uncontrolled Mutation
system state cannot change unpredictably
all changes must be scoped and intentional

4. Controlled Tool Access
tools are sandboxed
access is granted per action
access can be revoked instantly

Law
Control must exist at every step of execution.

🔁 Runtime Enforcement
During execution, Delegator:
validates each step against the Method
blocks deviation from defined actions
interrupts execution if constraints are violated
resets or restarts execution if needed

Law
Execution is continuously validated, not just initiated.

🧠 Relationship to Tools
Delegator governs:
EVO Browser
EVO Terminal
EVO Code (add-on)

Constraint
If the tool is not controlled, Alice cannot use it autonomously.

🔁 Relationship to Talents
Talents define allowed capabilities
Methods define execution steps
Delegator enforces both

Flow
```text Intent → Talent → Method → Delegator → Execution

## Related

^[source-materials/mirrors/doctrine/Connect - Delegator & Governance.md]
