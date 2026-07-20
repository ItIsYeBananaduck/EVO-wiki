# Hive Shared State Backbone

## Concept
All Hive devices share a single source of truth for Alice state.

## Rule / Mechanism
Hive must synchronize:
- chat history
- task manager state
- approved methods library
- Talents (metadata + snapshots)
- LoRA artifacts and versions
- logs/artifacts needed for continuity

## Why It Exists
Without shared state, “one Alice” breaks across devices.

## Implications
- State sync is mandatory
- Conflicts must resolve deterministically
- Devices can come and go without breaking continuity

## Links
- [[Single Alice Illusion]]
- [[LoRA Artifact Sync]]
- [[Hive Log Sync]]