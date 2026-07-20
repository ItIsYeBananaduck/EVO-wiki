# Task Actionability Gate

## Concept
Tasks are non-actionable by default. Execution requires an explicit authorization path.

## Rule / Mechanism
A task becomes actionable only if at least one is true:

1) Method Approval Path:
- Alice presents the method to the user
- The user approves the method

2) Talent Execution Path:
- Alice executes via a Talent permitted for that task

If neither is satisfied, the Delegator blocks tool access.

## Why It Exists
Prevents silent autonomy and ensures execution is governed.

## Implications
- Planning may occur without execution
- Execution must be explainable and auditable
- Tools are never accessible by default

## Links
- [[Delegator Tool Hostage Rule]]
- [[Method Approval Path]]
- [[Talent Execution Path]]