# Single Alice Illusion

## Concept
To the user, Alice is one continuous identity across all devices and apps.

## Rule / Mechanism
Hive and Swarm operations must be invisible by default.
The user should experience:
- one chat
- one task manager
- one memory/state
- one consistent personality

Implementation details (leases, nodes, shards) are not surfaced unless the user explicitly requests diagnostics.

## Why It Exists
The product promise is “one Alice,” not “distributed systems management.”

## Implications
- Default UI is minimal
- Debug/telemetry is optional and hidden
- Consistency beats transparency

## Links
- [[Hive Definition]]
- [[Execution Lease Rule]]