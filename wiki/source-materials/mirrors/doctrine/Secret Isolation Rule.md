# Secret Isolation Rule

## Concept
Sensitive values must be isolated from model-visible channels.

## Rule / Mechanism
Secrets must never appear in:
- chat transcript
- prompt context
- model outputs
- tool call logs

Secrets may exist only in:
- secure UI inputs
- encrypted vault storage
- short-lived execution memory

## Why It Exists
Model-visible secrets are effectively leaked.

## Implications
- Strong boundary between “chat” and “execution”
- Requires strict logging and telemetry filtering

## Links
- [[Secret Safe Protocols]]
- [[No Secret Echo Rule]]