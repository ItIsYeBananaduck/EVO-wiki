# AI Task Creation Is Non-Executable

## Concept
Alice may create tasks, but task creation does not imply authorization or execution.

## Rule / Mechanism
Tasks created by Alice enter a non-actionable state (e.g., Planned/AwaitingApproval).
They cannot execute until:
- the method is approved, or
- the method is a valid Talent

## Why It Exists
Allows autonomy in planning without autonomy in execution.

## Implications
- Users can review and approve proposed work
- Prevents silent automation
- Aligns with method/talent gating

## Links
- [[Task Actionability Gate]]
- [[Delegator State Machine MOC]]