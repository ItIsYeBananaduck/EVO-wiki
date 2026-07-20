# Device Linking via QR

## Concept
Devices join a user’s Hive by scanning a QR code (WhatsApp-style pairing).

## Rule / Mechanism
Pairing flow:
1) Primary device displays QR code containing a short-lived pairing payload
2) New device scans QR to request to join the Hive
3) Devices establish trust and exchange keys
4) New device begins initial sync (state + LoRAs as needed)

Pairing payload must be time-limited and single-use.

## Why It Exists
Fast, user-friendly linking without account friction.

## Implications
- Secure onboarding to the Hive
- Prevents accidental device joins
- Enables multi-device continuity

## Links
- [[Hive Pairing Trust Model]]
- [[Hive Shared State Backbone]]