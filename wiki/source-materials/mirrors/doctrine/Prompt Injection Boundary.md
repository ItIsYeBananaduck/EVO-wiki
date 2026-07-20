# Prompt Injection Boundary

## Concept
Untrusted content (especially web pages) must not be allowed to influence secret access.

## Rule / Mechanism
Web content is treated as untrusted input.
Secret access requires:
- a valid task authorization path
- a valid tool grant
- explicit token usage
and cannot be triggered by instructions found in web content.

## Why It Exists
Prompt injection is a primary exfiltration risk in browser automation.

## Implications
- Safety browser wrapper must isolate instructions from content
- Secret tokens cannot be resolved due to page content requests

## Links
- [[Safety Browser Protocol]]
- [[Secret Tokenization Rule]]
- [[Delegator Tool Hostage Rule]]
- [[Method Non-Deviation Rule]]
- [[Whitelisted Instruction Sources]]