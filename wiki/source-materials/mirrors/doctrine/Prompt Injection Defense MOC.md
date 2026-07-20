# Prompt Injection Defense MOC

## Purpose
Defines how the system safely handles untrusted input while remaining helpful and proactive.

This MOC explains **why prompt injection cannot cause execution** by design.

---

## Threat Model
Untrusted inputs include:
- Web pages
- Emails
- Documents
- Notifications
- App UI text
- Screen content
- Any non-chat, non-task-manager source

These sources may attempt to:
- Instruct Alice to act
- Bypass user intent
- Exfiltrate secrets
- Trigger tools

---

## Core Defense Strategy
Prompt injection is mitigated by **capability isolation**, not intent detection.

Untrusted content may inform planning, but **cannot cross the execution boundary**.

---

## Instruction Source Whitelisting
Only these sources may produce actionable instructions:
- [[Whitelisted Instruction Sources]]

Untrusted sources are informational only.

---

## Task Proposal From External Content
Untrusted content MAY:
- cause Alice to propose a task
- populate a draft method
- explain why the task might be useful

Untrusted content may NOT:
- approve a method
- grant tool access
- modify an approved method
- trigger execution

Related:
- [[AI Task Creation Is Non-Executable]]
- [[Untrusted Context Cannot Trigger Tools]]

---

## Delegator Enforcement
The Delegator enforces:
- [[Task Actionability Gate]]
- [[Delegator Tool Hostage Rule]]
- [[Method Non-Deviation Rule]]

Even if Alice is convinced, execution is impossible without authorization.

---

## Method Integrity
Once execution begins:
- Only the approved method (or immutable Talent snapshot) may be followed
- No external input may alter steps or tools

Related:
- [[Method Non-Deviation Rule]]
- [[Scoped Tool Grants]]

---

## Secrets Protection
Untrusted content cannot:
- request secrets
- access vault tokens
- influence secret resolution

Related:
- [[Secret Isolation Rule]]
- [[Prompt Injection Boundary]]

---

## User Experience Outcome
From the user’s perspective:
- Alice feels proactive and aware
- Nothing happens without consent
- All actions are reviewable
- Surprises are eliminated

---

## Architectural Rule
Untrusted content may suggest.
Only trusted channels may decide.
Only the Delegator may execute.