# Pairing Code Direction-Agnostic

## Concept
Pairing codes are direction-agnostic; either device may display or enter the code.

## Rule / Mechanism
A pairing code represents a short-lived pairing session.
Regardless of which device generates/displays it:
- the join request must be confirmed on the main device
- the code is single-use and time-limited
- trust is not granted until main-device approval

## Why It Exists
Supports devices without cameras and removes UX confusion about “who shows the code.”

## Implications
- Works across phone/tablet/desktop
- Maintains security guarantees
- Consistent pairing flow

## Links
- [[Main Device Root of Trust]]
- [[Pairing Code Protocol]]