# Whitelisted Instruction Sources

## Concept
Alice may only accept actionable instructions from approved sources.

## Rule / Mechanism
Actionable instructions are accepted only from:
- Chat (explicit user messages)
- Task Manager (tasks in authorized states)

All other sources are treated as untrusted context and cannot directly cause action.

## Why It Exists
Reduces the attack surface for prompt injection and malicious content.

## Implications
- Web pages, emails, notifications, and documents cannot directly instruct actions
- Those sources may be summarized, but not executed upon without user intent

## Links
- [[Prompt Injection Boundary]]
- [[Method Non-Deviation Rule]]
- [[Delegator Tool Hostage Rule]]