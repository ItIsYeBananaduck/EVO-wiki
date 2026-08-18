---
title: Pairing Code Direction-Agnostic
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Pairing Code Direction-Agnostic.md"]
updated: 2026-07-24
---

# Pairing Code Direction-Agnostic
[Pairing Code Direction-Agnostic](https://www.notion.so/33ec72bad01381cc9aaefcb7cde85b16)
Concept
Pairing codes are direction-agnostic; either device may display or enter the code.
Rule / Mechanism
A pairing code represents a short-lived pairing session. Regardless of which device generates/displays it: - the join request must be confirmed on the main device - the code is single-use and time-limited - trust is not granted until main-device approval
Why It Exists
Supports devices without cameras and removes UX confusion about “who shows the code.”
Implications
Works across phone/tablet/desktop
Maintains security guarantees
Consistent pairing flow
Links
[Main Device Root of Trust](https://www.notion.so/33ec72bad0138174b8caee1ba7b83dfb)
Pairing Code Protocol

## Related

^[{src_rel}]
