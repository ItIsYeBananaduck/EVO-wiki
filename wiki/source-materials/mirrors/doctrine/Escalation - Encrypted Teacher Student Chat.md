# Escalation – Encrypted Teacher↔Student Chat

Escalations are handled in a separate persistent support channel.

This channel is distinct from anonymous telemetry reporting.

---

## Two Planes

### Metrics Plane (Anonymous)
- rolling IDs
- aggregated metrics
- no names
- no chat content
- feeds TA/SE/DE/EVE reporting

### Support Plane (Named + Encrypted)
- teacher↔student encrypted chat (like trainer↔client)
- persistent thread for resolution
- used only for human intervention workflows

---

## Escalation Flow

1. SA detects template failure + high effort
2. SA asks student consent to start escalation thread
3. If approved, SA opens or reuses a concept-tag thread
4. SA sends:
   - concept tag
   - what was tried (template IDs)
   - optional student-approved summary
5. Teacher responds with guidance
6. Teacher may issue template unlock package in chat

---

## Template Unlock via Chat

Teacher sends signed unlock message:
- template_id(s)
- scope constraints (unit/subject)
- optional expiry
- teacher signature

Student device applies locally.
No mapping key required.
Telemetry IDs continue rolling independently.

---

## Guardrails

- escalation chat is not forwarded to EVE
- no analytics derived from chat content
- retention policy configurable by institution
- thread organization by concept tag