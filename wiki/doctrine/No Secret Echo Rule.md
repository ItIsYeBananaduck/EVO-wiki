---
title: No Secret Echo Rule
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/No Secret Echo Rule.md
updated: 2026-07-24
---

# No Secret Echo Rule
[No Secret Echo Rule](https://www.notion.so/33ec72bad013814892ccd1e5e72a397e)
Concept
The system must never re-display or repeat sensitive values.
Rule / Mechanism
Any secret entry UI masks input
Any internal logs redact or omit secrets
Any model response that attempts to output a secret is blocked/redacted
Why It Exists
Accidental echo is one of the most common secret leak paths.
Implications
Requires output filtering and log scrubbing
Builds user trust
Links
[Secret Isolation Rule](https://www.notion.so/33ec72bad013813e9eb6ead8af1141ad)
[Task Audit Log Minimum Fields](https://www.notion.so/33ec72bad01381aa9d87d0a77aa0cada)

## Related

^[source-materials/mirrors/doctrine/No Secret Echo Rule.md]
