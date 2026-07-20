# Silent Failure Preference

## Concept
When inference is unavailable or constrained, the system should degrade gracefully without noisy errors.

## Rule / Mechanism
If Alice cannot run:
- produce a basic deterministic output
- queue enhanced output for later if valuable
- avoid interrupting the user unless safety-critical

## Why It Exists
Failure modes should preserve trust and flow.

## Implications
- No broken-feeling UI
- Deferred intelligence is normal
- Users remain in control

## Links
- [[Deferred Response Strategy]]
- [[Non-Intrusive Guidance]]
- [[Energy-Aware Inference]]