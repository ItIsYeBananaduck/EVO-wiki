# Talent Revocation Rule

## Concept
Talents may be revoked, but never disabled or toggled.

## Rule / Mechanism
If a user no longer trusts a Talent:
- The Talent is revoked entirely
- All tasks referencing it become non-actionable
- Future use requires re-creation via method approval

Talents cannot be temporarily turned off.

## Why It Exists
Partial trust states are unsafe and ambiguous.

## Implications
- System behavior is deterministic
- Users understand consequences clearly
- Execution guarantees remain intact

## Links
- [[Talent Definition]]
- [[Delegator Tool Hostage Rule]]
- [[Task Actionability Gate]]