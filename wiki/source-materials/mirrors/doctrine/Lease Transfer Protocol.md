# Lease Transfer Protocol

## Concept
The execution lease transfers when the user switches active devices.

## Rule / Mechanism
Lease transfer requires:
- explicit user focus change (foreground / active interaction)
- revocation of tool grants on previous device
- confirmation of lease acquisition on new device

Transfers must be atomic.

## Why It Exists
Prevents partial execution across devices.

## Implications
- Clean handoff
- No split-brain execution

## Links
- [[Execution Lease Rule]]
- [[Scoped Tool Grants]]