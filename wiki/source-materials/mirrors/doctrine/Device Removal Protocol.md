# Device Removal Protocol

## Concept
Users can remove Hive devices from settings safely.

## Rule / Mechanism
Only the current main device may remove devices.
Removal:
- revokes trust credentials immediately
- blocks the removed device from reading hive state
- requires re-pairing to rejoin

## Why It Exists
Device removal is a security action and must be centrally enforced.

## Implications
- Lost/stolen devices can be cut off
- Hive remains secure

## Links
- [[Main Device Root of Trust]]
- [[Hive Security Settings Maintenance Mode]]