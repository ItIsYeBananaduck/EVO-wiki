---
title: Talent Classes and Governance Boundaries
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Talent Classes and Governance Boundaries.md
updated: 2026-07-24
---

# Talent Classes and Governance Boundaries
## Purpose

Define the different classes of Talents and how governance is applied across execution domains.

---

## Core Principle

> All Talents are governed.

> Governance location depends on execution surface.

---

## Talent Classes

### App Talents (Closed Execution Domains)

App Talents operate entirely within a closed execution environment and do not interact with external tools or systems.

Domains that use App Talents:

- EVOmind
- EVOtraining

App Talents:

- execute through internal application logic
- operate on internal state and data only
- do not call external APIs, files, or system resources
- do not require Delegator governance for execution

All behavior is governed by:

- application-level constraints
- bounded execution rules
- Talent immutability and validation rules

> App Talents are fully governed, but their governance is enforced internally by the application rather than by Delegator.

---

### Promoted Talents

Promoted Talents are Methods that have been validated and explicitly preserved by the user.

They:

- originate from repeated successful Method execution
- require user consent for preservation
- are stable and bounded
- reduce repeated reasoning and approval overhead

Promoted Talents may operate across domains depending on execution surface.

---

### Trained Talents

Trained Talents are explicitly taught by the user or constructed through guided workflows.

They:

- may originate from user demonstration
- may include structured or semi-structured logic
- must still comply with execution governance rules
- are not automatically trusted without validation

---

## Delegator Scope Boundary

Delegator governs only execution that interacts with:

- external tools
- file systems
- browsers or terminals
- APIs or third-party systems
- shared or cross-domain resources

Delegator does not govern:

- internal cognitive systems
- internal state updates
- closed-domain application logic

> Delegator is the enforcement layer for external execution.

> App Talents are the enforcement layer for internal execution.

---

## Final Principle

> Internal systems are governed by application logic.

> External systems are governed by Delegator.

Both are required for a complete, safe execution model.

---

## Related Notes

- [Talent Promotion Rule](https://www.notion.so/33ec72bad013814389d2efd20e39c2c6)
- [Task Chain Definition](https://www.notion.so/343c72bad01381d1a3e3f35f210f82d9)
- [Execution Model: Intent → Effect → Execution](https://www.notion.so/343c72bad01381498ea5e9e5312270df)
- [Delegator Doctrine: Execution Authority](https://www.notion.so/343c72bad01381ef9ad0d496a384113b)
- [EVOconnect Talent Model](https://www.notion.so/33dc72bad0138188bcf7e7b995b3ac5f)
^[source-materials/mirrors/doctrine/Talent Classes and Governance Boundaries.md]
