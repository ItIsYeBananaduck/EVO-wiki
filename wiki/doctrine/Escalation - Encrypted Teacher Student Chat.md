---
title: Escalation - Encrypted Teacher Student Chat
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Escalation - Encrypted Teacher Student Chat.md
updated: 2026-07-24
---

# Escalation - Encrypted Teacher Student Chat
Escalation – Encrypted Teacher↔Student Chat
Escalations are handled in a separate persistent support channel.
This channel is distinct from anonymous telemetry reporting.

Two Planes
Metrics Plane (Anonymous)
rolling IDs
aggregated metrics
no names
no chat content
feeds TA/SE/DE/EVE reporting
Support Plane (Named + Encrypted)
teacher↔student encrypted chat (like trainer↔client)
persistent thread for resolution
used only for human intervention workflows

Escalation Flow
SA detects template failure + high effort
SA asks student consent to start escalation thread
If approved, SA opens or reuses a concept-tag thread
SA sends:
concept tag
what was tried (template IDs)
optional student-approved summary
Teacher responds with guidance
Teacher may issue template unlock package in chat

Template Unlock via Chat
Teacher sends signed unlock message: - template_id(s) - scope constraints (unit/subject) - optional expiry - teacher signature
Student device applies locally. No mapping key required. Telemetry IDs continue rolling independently.

Guardrails
escalation chat is not forwarded to EVE
no analytics derived from chat content
retention policy configurable by institution
thread organization by concept tag

## Related

^[source-materials/mirrors/doctrine/Escalation - Encrypted Teacher Student Chat.md]
