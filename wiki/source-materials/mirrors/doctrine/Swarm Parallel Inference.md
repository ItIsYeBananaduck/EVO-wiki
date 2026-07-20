# Swarm Parallel Inference

## Concept
When a prompt is too compute-taxing for any single device, the lease holder activates the Swarm.

## Rule / Mechanism
Swarm mode:
- splits the overall inference into multiple sub-tasks
- assigns sub-tasks to Hive nodes based on capability
- runs inference in parallel across nodes
- returns partial results to the lease holder
- lease holder assembles a single coherent user-facing output

## Why It Exists
Enables heavy reasoning without requiring cloud compute while keeping the system local-first.

## Implications
- Parallelism reduces latency and thermal load
- Lease holder remains responsible for final output
- Nodes remain bounded to assigned sub-tasks

## Links
- [[Swarm Task Sharding]]
- [[Swarm Work Ticket]]
- [[Single Executor Guarantee]]