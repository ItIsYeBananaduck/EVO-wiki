# Hive Definition

## Concept
The Hive is the collection of Alice instances running locally on a user’s devices.

## Rule / Mechanism
Each device runs a local Alice model.
All Hive members:
- share the same chat
- share the same tasks
- share the same state

At any time, only one device holds the execution lease.

## Why It Exists
Enables seamless multi-device continuity without concurrent execution risk.

## Implications
- Alice feels omnipresent but acts singularly
- No duplicated or conflicting actions
- Local-first execution remains safe

## Links
- [[Execution Lease Rule]]
- [[Single Executor Guarantee]]