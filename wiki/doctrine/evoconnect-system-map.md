---
title: EVOconnect — System Map
type: concept
tags: [evo, connect, execution, terminal, browser, swarm, integration]
sources:
  - source-materials/mirrors/doctrine/EVOconnect — System Map.md
updated: 2026-07-20
---

# EVOconnect — System Map

## Purpose
Connect is the control plane over EVO execution surfaces: browser, terminal, desktop, mobile.

## Execution Surfaces
- EVOterminal — local shell execution
- EVObrowser — browser automation
- EVOvault — secure secret storage
- Task Manager — agent supervision layer

## Multi-Agent Runtime
Connect governs multi-agent orchestration with explicit approval gates, bounded talents, and runtime enforcement.

## Context Boundary
Connect owns control-plane trust boundaries. It does not silently mutate training or journaling data owned by other apps.

## Related
- [[EVO Architecture Bible]]
- [[Connect — Task Control Plane]]
- [[Governance & Authority Map]]

^[source-materials/mirrors/doctrine/EVOconnect — System Map.md]
