# Task Transparency Retention

## Concept
Completed and failed tasks are retained for a limited time to provide transparency into Alice’s behavior.

## Rule / Mechanism
After execution, task records (including method reference, tool call log, and outputs summary) remain visible to the user for a defined retention period.

After retention expires:
- details may be pruned or summarized
- user may optionally export before deletion

## Why It Exists
Transparency builds trust and makes autonomy auditable.

## Implications
- Users can review what happened
- Debugging is easier
- Storage cost remains controlled

## Links
- [[Delegator State Machine MOC]]
- [[Silent Failure Preference]]