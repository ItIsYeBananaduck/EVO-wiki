# Main Device Root of Trust

## Concept
The Hive has a designated main device that serves as the root of trust for pairing.

## Rule / Mechanism
Only the main device may finalize pairing:
- approve a new device joining the Hive
- issue trust credentials/keys for sync
- revoke devices from the Hive

Other devices may initiate pairing requests but cannot grant trust.

## Why It Exists
Prevents unauthorized devices from joining and simplifies the user mental model.

## Implications
- One clear approval point
- Strong security boundary
- Consistent pairing UX

## Links
- [[Hive Pairing Trust Model]]
- [[Multi-Modal Pairing]]