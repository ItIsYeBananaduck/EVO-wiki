# Hive Pairing Trust Model

## Concept
Hive devices must trust each other through explicit pairing.

## Rule / Mechanism
Pairing establishes:
- device identity
- encryption keys for sync transport
- authorization to access shared state and LoRAs

Unpaired devices cannot:
- read hive chat
- read tasks
- receive work tickets
- access synced LoRAs/logs

## Why It Exists
Multi-device sync is a major security boundary.

## Implications
- Pairing is required for any participation
- Devices can be revoked from the Hive

## Links
- [[Device Linking via QR]]
- [[Secret Isolation Rule]]