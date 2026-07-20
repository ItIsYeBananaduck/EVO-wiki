  

# Safety Browser Protocol

## Concept
Sensitive workflows should be executed inside a controlled browser sandbox with a privacy wrapper.

## Rule / Mechanism
- Alice performs actions in a controlled browser environment
- The wrapper detects sensitive fields and prevents secrets from appearing in model-visible text
- User can intervene at any time
- The browser layer can request secrets via the vault using tokens/handles

## Why It Exists
Many real tasks require web workflows where secrets may appear on-screen or in form fields.

## Implications
- Prevents prompt injection via web content from exfiltrating secrets
- Makes “AI does things on my behalf” feasible and auditable

## Links
- [[Advanced Vault Protocol]]
- [[Secret Isolation Rule]]
- [[Prompt Injection Boundary]]