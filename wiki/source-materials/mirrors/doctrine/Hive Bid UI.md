  

# Hive Bid UI

## Concept
Hive bidding must be user-legible but not disruptive.

## Rule / Mechanism
When a bid request is opened:
- show a non-blocking "Hive Assist" banner on the lease device
- allow expand to a "Bid Panel" displaying devices and bids

Bid Panel must show per device:
- device name + type (Phone/Tablet/Desktop)
- status (Active/Idle, Charging, Thermal)
- bid summary (ETA, energy, confidence)
- reason (e.g., "Model already warm", "On charger", "High compute")

Selection:
- default auto-select best bid (per scoring rule)
- user can override selection
- user can cancel delegation

Completion:
- show "Hive Result Received" toast
- result is inserted into the chat/task as an attributed artifact

## Why It Exists
Users should understand why work moved to another device, without being forced to micromanage.

## Implications
- Transparency
- Optional manual control
- Predictable UX

## Links
- [[Hive Bid Scoring Rule]]
- [[External Task Proposal Attribution]]