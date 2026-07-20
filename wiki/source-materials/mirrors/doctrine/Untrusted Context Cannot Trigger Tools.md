# Untrusted Context Cannot Trigger Tools

## Concept
Untrusted content must never directly trigger tool access.

## Rule / Mechanism
Information from untrusted sources (web content, third-party text, external prompts) may:
- inform planning
- be displayed to the user
- propose a task or method

But it may not:
- grant tool access
- alter an approved method
- initiate execution

## Why It Exists
Most prompt injection attempts rely on getting the model to “just do it.”

## Implications
- External content can only influence action via explicit user approval
- Delegator blocks any attempt to translate untrusted context into execution

## Links
- [[Whitelisted Instruction Sources]]
- [[Task Actionability Gate]]
- [[Delegator Tool Hostage Rule]]