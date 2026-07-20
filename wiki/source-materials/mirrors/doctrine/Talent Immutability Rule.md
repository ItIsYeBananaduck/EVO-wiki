# Talent Immutability Rule

## Concept
Talents are immutable once created.

## Rule / Mechanism
A Talent’s method definition may not be edited after promotion.

If behavior must change:
- The Talent is revoked
- A new method is defined
- The method must be re-approved and re-promoted

## Why It Exists
Editing Talents would silently change trusted behavior and violate safety guarantees.

## Implications
- Talents are stable and predictable
- Trust decisions remain valid over time
- Versioning is explicit through re-creation, not mutation

## Links
- [[Talent Definition]]
- [[Talent Promotion Rule]]
- [[Talent Revocation Rule]]
- [[No Talent Toggle Rule]]