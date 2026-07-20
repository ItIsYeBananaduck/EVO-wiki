# Hive Device Presence Icons

## Concept
Hive and Swarm status is represented to the user via small glowing device icons.

## Rule / Mechanism
Each device in the Hive is represented by a user-configurable icon (with defaults per device type).
Icon colors reflect status:

- Green: device ON + available
- Red: device ON + unavailable (constraints prevent participation)
- Blue: device is the current lease holder (active executor)
- Blue flashing: Swarm is active (multiple devices running inference in parallel)
- Gray: device OFF

Icons must be visible in relevant views (chat/task context) without disrupting the user.

## Why It Exists
Transparency builds trust and makes distributed compute feel legible.

## Implications
- Users understand where work is happening
- Users can diagnose “why it’s slow”
- Supports user mental model of the Hive/Swarm

## Links
- [[Execution Lease Rule]]
- [[Swarm Parallel Inference]]