---
title: Bunker Onboarding and First-Run Trust Model
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Bunker Onboarding and First-Run Trust Model.md"]
updated: 2026-07-24
---

# Bunker Onboarding and First-Run Trust Model
## Purpose

Define how EVO introduces Bunkers and protected storage to the user before Alice is initialized, ensuring that trust boundaries are established prior to any AI interaction.

This model ensures users understand:

- what Alice can access
- what remains private
- how to protect sensitive data
- how temporary access works
- how authentication governs boundary changes

---

## Core Principle

Alice should not begin full operation until the user has been given the opportunity to define trust boundaries.

Trust is established before intelligence is activated.

---

## Why This Matters

If Alice is initialized first, users experience:

- uncertainty about what the AI can see
- reactive privacy management
- reduced trust

If Bunkers are introduced first, users experience:

- control
- clarity
- confidence in system boundaries

---

## First-Run Trust Zones

During onboarding, the system should introduce three clear zones:

### 1. Alice Workspace

- Fully accessible by Alice
- Used for normal interaction, organization, and execution

### 2. Bunkers (Protected Areas)

- Not accessible by Alice through normal operations
- Hidden from browsing, search, and summarization
- Fully user-controlled

### 3. Temporary Access

- User can grant limited access to specific protected content
- Access is scoped, time-bound, and authenticated

---

## Onboarding Flow

### Step 1 — Introduce Trust Model

Explain simply:

- “Alice can help with your files and tasks”
- “You can protect anything you want from Alice”
- “Protected areas are called Bunkers”

### Step 2 — Select Protected Areas

Allow the user to optionally select:

- local folders
- cloud folders
- project directories
- personal archives
- sensitive system locations

System creates initial Bunkers from these selections.

### Step 3 — Create Default Bunkers

System may suggest:

- Documents / Personal
- Financial / Sensitive
- Work Archives
- Custom user-defined folders

All Bunkers are registered and enforced by the system.

### Step 4 — Explain Temporary Access

Tell the user:

- “Alice cannot access Bunker contents unless you allow it”
- “You can temporarily unlock specific files or tasks”
- “Access is automatically revoked after the session”

### Step 5 — Bind to System Authentication

All Bunker boundary changes require:

- system password
- biometric authentication (Face ID, Touch ID, Windows Hello)
- OS-native secure confirmation

This applies to:

- creating Bunkers
- modifying Bunkers
- granting temporary access
- removing protection
- changing inheritance rules

### Step 6 — Confirm Setup

User confirms:

- protected areas
- understanding of access model

### Step 7 — Initialize Alice

Only after trust boundaries are defined:

- Alice initializes
- Delegator enforces protection rules immediately
- Alice operates only within allowed surfaces

---

## Inheritance During Onboarding

When a user protects a container:

- all children inherit protection by default

Examples:

### Folder

- protect `/Users/Phil/Documents`
- all subfolders and files are protected

### Project

- protect Project A
- phases, tasks, subtasks inherit protection

This avoids:

- repetitive configuration
- accidental exposure

---

## Temporary Access During Use

When Alice needs protected content:

### Flow

1. Alice detects need
2. Alice requests access
3. User approves
4. User authenticates
5. Delegator creates scoped session
6. Access granted temporarily
7. Session expires automatically

---

## Resource Management Case

If protected content is consuming system resources:

Alice may:

- detect pressure (RAM, CPU, disk)
- identify protected resource usage

Alice may NOT:

- inspect contents
- manipulate protected processes

Correct behavior:

- ask user for permission
- require authentication
- receive scoped access session
- perform only approved actions

---

## User Experience Goals

The onboarding must feel:

- simple
- empowering
- not technical
- not overwhelming

Avoid technical terms like:

- metadata
- Delegator
- path registry
- enclave

Use language like:

- protected areas
- Bunkers
- temporary unlock
- secure access

---

## Security Model

### Default

- everything in Bunkers is inaccessible to Alice

### Exception

- access only via authenticated session

### Expiration

- access automatically revoked

### No Silent Downgrades

- protected content never becomes unprotected without explicit user action

---

## Relationship to System Architecture

This onboarding configures:

- Bunker metadata
- protected path registry
- Delegator enforcement rules
- access session system
- inheritance behavior

All future system behavior depends on this initial configuration.

---

## Design Rule

Alice should never operate with unknown access boundaries.

The system must always know:

- what is visible
- what is hidden
- what requires permission

before Alice begins operation.

---

## Summary

The Bunker Onboarding and First-Run Trust Model ensures:

- users define privacy boundaries before AI activation
- protected areas are clearly understood
- temporary access is safe and controlled
- authentication secures all boundary changes
- Alice operates only within explicitly allowed surfaces

This establishes trust as a foundational property of the EVO system.

---

## Related Notes

- [Bunker Model — Protected User and System Storage Containers](https://www.notion.so/343c72bad0138186af70ec9b2ce2ba9f)
- [Bunker Access Session Model](https://www.notion.so/343c72bad01381788004c1ec4b0b695d)
- [Protection Classification and Inheritance Model](https://www.notion.so/343c72bad01381928bd0fb9c94164e20)

## Doctrine Update — 2026-05-01 — Bunker Visibility in Connect Library

Bunkers are visible to the user in Connect Library as protected zones.

This note’s prior “hidden” language refers to Alice’s operational visibility, not the user’s view.

Revised model:

- The user can see Bunker containers in Connect Library.
- Alice may know a Bunker exists as a protected zone.
- Alice cannot browse, search, summarize, inspect, organize, or modify Bunker contents through normal execution paths.
- Access requires a scoped, expiring, authenticated session.
- Bunker contents are excluded from Alice’s default operational index.

Canonical reference:
EVOconnect — Connect Library and Bunker Visibility Model
