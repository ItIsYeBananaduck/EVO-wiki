---
title: Environment Integrations
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Environment Integrations.md"]
updated: 2026-07-24
---

# Environment Integrations
Purpose
Allow Alice to interact with tools that do not support direct API or OAuth integration.

Examples
Claude web app
Local development tools
IDEs
Browser-based platforms
CLI tools

Why This Exists
Some systems: - do not expose APIs- restrict OAuth usage- are designed for human interaction
Environment integrations allow Connect to still leverage them.

Interaction Methods
EVOterminal execution
Internal browser interaction
User-assisted workflows
Semi-automated flows

Modes
Assisted Mode
Alice prepares action
User executes or confirms
Semi-Automated Mode
Alice executes within allowed scope
User approves sensitive actions

Constraints
No direct account hijacking
No hidden automation
No bypassing provider rules

Output Handling
Results captured into task
Linked to escalation
Logged for audit

[Philosophy](https://www.notion.so/33ec72bad0138193a0bbce9f89d79395)
Environment integrations treat tools as: - external environments- not native system components

## Related
