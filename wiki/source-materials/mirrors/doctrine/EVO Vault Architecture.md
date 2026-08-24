---
title: EVO Vault Architecture
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/EVO Vault Architecture.md"]
updated: 2026-07-24
---

﻿EVO Vault Architecture
Learning from Varlock While Building a User-First AI Vault

Status
Product Architecture Thesis

Purpose

EVO Vault is the trust and identity layer of the EVO ecosystem.

Its purpose is not simply to store secrets.

Its purpose is to allow Alice to accomplish work while revealing the minimum amount of user information necessary.

Unlike traditional password managers or secret stores, EVO Vault manages knowledge, identity, permissions, capabilities, and disclosure policies rather than merely encrypted values.

Inspiration

Varlock demonstrates an important architectural principle:

An AI system can understand the existence and purpose of a secret without receiving the secret itself.

Varlock accomplishes this primarily through managed variables and secure injection.

EVO adopts this philosophy but expands it into a generalized user privacy architecture.

Instead of protecting only developer secrets, EVO Vault protects every category of user-owned information.

Core Philosophy

Traditional AI systems become more capable by learning more about the user.

EVO becomes more capable by requiring less knowledge about the user.

This is the guiding principle of EVO Vault.

Alice should never possess sensitive information unless possession is genuinely required to complete the user's objective.

Whenever possible:

• Alice knows that information exists.
• Alice knows what the information represents.
• Alice requests permission to use it.
• The system performs the action.
• Alice receives only the outcome.

Knowledge is separated from execution.

User-Owned Data

Everything stored inside EVO Vault belongs to the user.

Examples include:

• identity information
• addresses
• phone numbers
• email addresses
• payment methods
• government identifiers
• passwords
• passkeys
• API keys
• authentication tokens
• healthcare information
• financial information
• business credentials
• personal preferences
• automation variables

The vault exists for the user's benefit.

Not for model training.

Not for analytics.

Not for advertising.

Privacy Is Not Binary

Traditional systems classify information as either public or secret.

EVO introduces progressive disclosure.

Every vault object has its own disclosure policy.

Examples include:

• Can Alice know this exists?
• Can Alice read the plaintext?
• Can Alice use it without seeing it?
• May a local model receive it?
• May a cloud model receive it?
• Which domains may receive it?
• Does every use require approval?
• Does approval expire?
• Should the value ever appear in logs?
• Should the value ever appear inside model context?

Each user decides how much disclosure they are comfortable with.

Capability Over Possession

The central design principle is:

Alice requests capabilities—not secrets.

Instead of requesting, “Give me the user's address,” Alice requests, “Fill the shipping address.”

Instead of requesting, “Give me the GitHub token,” Alice requests, “Authenticate this Git operation.”

The trusted runtime resolves the request.

Alice receives confirmation rather than the secret itself.

Vault Objects

EVO Vault stores structured objects instead of raw strings.

Examples include:

• Home Address
• Work Address
• Credit Card
• Passport
• GitHub Credential
• Insurance Card
• Wi-Fi Credential
• Business Identity
• Personal Identity

Each object contains:

• encrypted values
• metadata
• disclosure policy
• execution policy
• audit history
• ownership information

This provides significantly richer policy enforcement than traditional environment variables.

Disclosure vs. Execution

Two independent questions determine access.

Disclosure

Who may know the value?

Execution

Who may use the value?

These are intentionally separate.

Alice may be allowed to use a credential while remaining unable to view its contents.

Trust Tiers

Users should not configure dozens of security switches.

Instead, EVO should provide understandable trust tiers.

Convenience

Minimal prompts. Trusted destinations execute automatically.

Balanced

Sensitive operations require periodic confirmation. Routine operations execute automatically.

Strict

Most sensitive actions require authentication. New destinations always require approval.

Maximum Security

Every protected action requires explicit user authentication.

Users may customize these policies if desired.

Authentication Policies

Authentication should be configurable per vault object.

Examples:

• once during setup
• on first use
• on first use per destination
• every session
• every execution
• only after policy changes
• only after unusual behavior is detected

Different users have different security expectations.

EVO should accommodate both convenience and maximum privacy.

Passkeys and Trusted Execution

Passkeys represent one of the strongest use cases for EVO Vault.

The passkey never leaves secure hardware.

Alice requests authentication.

The Delegator evaluates policy.

If required, the user authenticates using biometrics or device authentication.

The operating system completes the passkey exchange.

Alice learns only whether authentication succeeded.

The credential remains sealed.

Trusted Injection

Some information should be inserted without exposing its plaintext to Alice.

Examples:

• addresses
• payment methods
• passwords
• API tokens
• personal identifiers

The flow becomes:

Alice identifies the required field.

Alice requests a vault capability.

The Delegator evaluates policy.

A trusted browser or operating system component injects the value.

Alice receives success or failure.

The model never requires direct access to the protected value.

Browser Integration

Alice's browser should distinguish between:

• information Alice reads
• information Alice requests
• information the browser injects

Sensitive autofill operations should occur inside trusted browser components rather than through model-generated keystrokes whenever possible.

Minimum Necessary Context

Whenever Alice delegates work to another model, she should provide only the context necessary to complete the task.

External models should receive:

• objectives
• constraints
• relevant knowledge
• authorized capabilities

They should not automatically receive:

• complete user history
• entire journals
• complete vault contents
• unrelated personal information

Every delegation follows the principle of minimum necessary disclosure.

Context Minimization

The purpose of delegation is not to expose the user's life.

It is to accomplish a specific task.

Whenever possible, Alice should translate user information into generalized context before delegation.

The external model receives only what it needs.

Nothing more.

User Trust

Most AI companies become more capable by collecting more user data.

EVO seeks to become more capable through better architecture.

The user's knowledge remains the user's knowledge.

The user's vault remains the user's vault.

Model upgrades never require the user to surrender additional personal information.

Relationship to Alice

Alice is the steward of the vault.

She is not its owner.

She cannot grant authority beyond what the user has delegated.

She cannot permanently expose protected information simply because another model requests it.

Every action ultimately traces back to user-defined policy enforced by the Delegator.

Relationship to the Delegator

The Delegator is the enforcement authority.

It evaluates:

• disclosure rules
• execution policies
• authentication requirements
• destination trust
• capability requests
• audit logging

Alice requests.

The Delegator decides.

The trusted runtime executes.

Design Principles

• The user owns their information.
• Alice should know as little as possible while accomplishing as much as possible.
• Capabilities are preferable to secrets.
• Disclosure and execution are separate permissions.
• Authentication is policy-driven.
• External models receive only the minimum necessary context.
• Sensitive operations should occur inside trusted system components whenever possible.
• Models are replaceable.
• User trust is more valuable than user data.

Product Thesis

EVO Vault is not designed to hide information from the user.

It is designed to prevent unnecessary exposure of user information—even to Alice herself.

The objective is not simply secure storage.

The objective is to make privacy the default operating model of an intelligent assistant.

By treating user information as user-owned capabilities instead of model-owned knowledge, EVO enables powerful AI assistance without requiring users to surrender control of their digital lives.

## Related
