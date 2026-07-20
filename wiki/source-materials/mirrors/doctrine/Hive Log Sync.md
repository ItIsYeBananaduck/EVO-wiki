# Hive Log Sync

## Concept
Hive devices share logs/artifacts required for continuity, not user-facing debugging.

## Rule / Mechanism
Hive synchronizes:
- task outcomes (completed/failed)
- exercise/workout logs (for EVOtraining)
- method approval history (for promotion thresholds)
- minimal provenance needed for correctness

Logs are not surfaced to the user by default.

## Why It Exists
Cross-device continuity requires shared history even if the user never sees it.

## Implications
- Retention can exist without UI exposure
- Logs must avoid plaintext secrets
- Sync must be conflict-safe

## Links
- [[Task Transparency Retention]]
- [[Secret Isolation Rule]]
- [[Hive Shared State Backbone]]