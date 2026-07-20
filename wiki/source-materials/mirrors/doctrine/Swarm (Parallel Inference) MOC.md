# Swarm (Parallel Inference) MOC

## Purpose
Defines how the lease holder distributes heavy inference across Hive devices in parallel and merges results.

---

## Core Concepts
- [[Swarm Parallel Inference]]
- [[Swarm Activation Criteria]]

---

## Sharding & Tickets
- [[Swarm Task Sharding]]
- [[Swarm Work Ticket]]

---

## Merge & Output
- [[Swarm Merge Rule]]

---

## UI
- [[Hive Device Presence Icons]]
- [[Hive Icon Status Mapping]]

---

## Safety & Boundaries
- [[Single Executor Guarantee]]
- [[No Tool Access During Planning]]
- [[Secret Isolation Rule]]
- [[Whitelisted Instruction Sources]]
- [[Invisible Compute Orchestration]]
- [[Online Nodes Requirement]]
- [[Offline Lease Holder Rule]]

---

## Assignment, Resilience & Merge Control
- [[Shard-to-Device Assignment Heuristics]]
- [[Swarm Shard Retry Policy]]
- [[Shard Confidence Weighting]]