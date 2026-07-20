# No Secret Echo Rule

## Concept
The system must never re-display or repeat sensitive values.

## Rule / Mechanism
- Any secret entry UI masks input
- Any internal logs redact or omit secrets
- Any model response that attempts to output a secret is blocked/redacted

## Why It Exists
Accidental echo is one of the most common secret leak paths.

## Implications
- Requires output filtering and log scrubbing
- Builds user trust

## Links
- [[Secret Isolation Rule]]
- [[Task Audit Log Minimum Fields]]