# Method Approval Path

## Concept
User approval of a method authorizes task execution.

## Rule / Mechanism
Alice must present the task’s method to the user, including:
- Steps
- Tools required
- Expected outputs
- Risks or side effects (if applicable)

Upon explicit user approval:
- The task becomes executable
- Tool access is granted in a scoped manner

## Why It Exists
User intent must be explicit before automation.

## Implications
- Approval is task-scoped unless promoted
- Approval can be revoked
- Approved methods may later qualify for Talent promotion

## Links
- [[Task Actionability Gate]]
- [[Method Is Mandatory]]
- [[Scoped Tool Grants]]