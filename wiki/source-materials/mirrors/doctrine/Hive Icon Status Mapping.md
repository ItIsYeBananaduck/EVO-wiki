# Hive Icon Status Mapping

## Concept
Device icon states must be derived deterministically from device capability + lease + swarm activity.

## Rule / Mechanism
- Gray if device is offline
- Blue if device holds the lease
- Blue flashing if Swarm mode is active AND device is participating
- Green if device is online and can accept delegated work tickets
- Red if device is online but cannot accept work tickets (thermal, low battery, no model, user disallowed, etc.)

When Swarm mode is active:
- all participating devices flash blue
- non-participating online devices remain green (available) or red (unavailable)
- offline devices remain gray
- lease holder remains blue and also flashes while participating
## Why It Exists
Prevents ambiguous UI state.

## Implications
- Consistent visuals
- Easier debugging

## Links
- [[Hive Device Presence Icons]]
- [[Hive Capability Advertisement]]