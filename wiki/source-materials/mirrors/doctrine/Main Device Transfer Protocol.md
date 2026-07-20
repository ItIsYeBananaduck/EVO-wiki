# Main Device Transfer Protocol

## Concept
Users may change the main device via an explicit transfer handshake.

## Rule / Mechanism
Main device transfer requires:
- initiation on the target device
- explicit approval on the current main device
- atomic role swap (no overlap)

During transfer:
- active pairing sessions are invalidated
- only one device may be main at any time

## Why It Exists
Prevents split-brain trust and keeps pairing authority unambiguous.

## Implications
- Users can upgrade phones safely
- Clear authority boundary
- Stable multi-device UX

## Links
- [[Main Device Root of Trust]]
- [[Hive Security Settings Maintenance Mode]]