---
title: Work Journal System
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/Work Journal System.md"]
updated: 2026-07-24
---

# Work Journal System
## Purpose

Define the work-knowledge memory layer governed by Connect.

## Scope

The Work Journal stores what Alice learns about doing work better, including:

- productive workflow patterns
- agent behavior observations
- prompt and issue structure lessons
- execution failures
- reusable heuristics
- reasoning mistakes and corrections

## Allowed Sources

- Connect execution logs
- task outcomes
- GitHub issue and PR patterns
- external agent observations
- user correction events when Alice inferred incorrectly

## Write Rules

Alice should create a work journal entry when:

- a workflow pattern appears useful and repeatable
- an agent repeatedly succeeds or fails under similar conditions
- a correction reveals a reasoning flaw worth remembering
- a task outcome suggests a reusable heuristic

Alice should not write:

- noise from a single inconclusive run
- raw logs copied without interpretation
- lessons that are too vague to reuse

## Relationship to Connect

Connect owns the Work Journal because Connect is the execution and observation layer.

Other apps may benefit from its lessons, but Connect governs how those lessons are formed, refined, and promoted.

## Principle

The Work Journal is how Alice learns how to work better, not who the user is.

## Related
