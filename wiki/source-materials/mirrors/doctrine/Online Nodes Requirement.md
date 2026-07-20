# Online Nodes Requirement

## Concept
Non-lease Hive nodes must be online to participate in Hive/Swarm compute and synchronization.

## Rule / Mechanism
For delegation/Swarm participation:
- nodes must have internet connectivity
- nodes must be reachable for ticket dispatch and result return
- nodes must sync LoRAs and state

## Why It Exists
Swarm compute and shared state require connectivity between devices.

## Implications
- Offline nodes are treated as unavailable
- Sync resumes when nodes return online

## Links
- [[Hive Shared State Backbone]]
- [[Swarm Work Ticket]]