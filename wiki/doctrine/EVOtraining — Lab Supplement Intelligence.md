---
title: EVOtraining — Lab Supplement Intelligence
type: concept
tags: [evo, evotraining]
sources: []
origin: wiki-native — authored in this wiki, no upstream mirror
updated: 2026-07-23
---
# EVOtraining — Lab Supplement Intelligence

## Purpose

EVOtraining Lab is the experimental and advanced-performance branch of EVOtraining.

The goal is a system where Alice gradually learns how training variables, recovery variables, nutrition variables, and supplement variables interact over time — both for the individual user and across the aggregate ecosystem.

---

## Foundation: SmartScan Reuse

SmartScan's dynamic field extraction and schema-flexible parsing (originally for nutrition labels) is reused for Training Lab. Supplement labels are inconsistent across brands — ingredients differ, stimulant blends differ, serving structures differ. Training Lab therefore uses flexible ingestion rather than rigid schemas.

---

## Daily Compliance Journal Model

Users scan supplements once using SmartScan (pre-workouts, intra-workouts, hydration mixes, post-workouts, recovery supplements, sleep aids, vitamins, nootropics, and other performance compounds).

After initial scanning, users do not repeatedly rescan. Instead, EVOtraining periodically asks what was actually consumed — pre-workout, post-workout, or as an end-of-day check — using a lightweight checkbox interaction (select individual supplements, deselect skipped items, or press "taken all").

The key metric is **consistency tracking**.

---

## Correlation Engine

Over time, Alice correlates:
- workout quality, fatigue, strain, recovery
- HRV, sleep, soreness, pump
- performance progression, perceived exertion, recovery speed
- nutrition intake and supplement usage patterns

The system becomes significantly more valuable when users change only one variable at a time (replacing one pre-workout, adding creatine, removing caffeine, adjusting hydration compounds, changing carb timing).

This allows Alice to observe probable relationships between supplementation, recovery, energy, and performance outcomes — contributing both to the personal adapter and aggregate learning systems.

---

## Aggregate Learning Scope

Aggregate learning is not medical advice or autonomous supplement prescribing.

Alice provides informed contextual guidance when users ask questions. Example: a user reporting low energy mid-workout receives an evaluation of recovery, sleep, nutrition, hydration, workout intensity, stimulant usage, historical supplement responses, and aggregate outcomes.

If nutrition and recovery appear acceptable, Alice may determine the issue is likely supplementation-related and reference aggregate preference and outcome patterns (which pre-workouts users respond well to, which ingredients correlate with perceived energy improvements, which stimulant levels are commonly tolerated).

---

## Connect-Assisted Research (V3+)

In V2, aggregate intelligence is local-to-ecosystem (generated from user observations and adapter learning). No internet browsing is required.

In V3+, Connect changes this. Example user query: "What's the cheapest but most effective pre-workout for me?"

EVOtraining delegates the request through Connect → the delegator constrains the query → Alice performs bounded internet retrieval → results are returned as structured informational output only.

The delegator's responsibility:
- scope enforcement and action restriction
- query integrity and task adherence

EVOtraining safely inherits Connect's capabilities without embedding unrestricted internet behavior.

---

## Long-Term Direction

Future systems under Training Lab:
- supplement comparison, ingredient education, aggregate effectiveness analysis
- budget-aware recommendations and stack optimization
- pose-estimation-assisted experimentation and movement analysis
- advanced recovery correlations and experimental adaptation models
- hybrid cloud-assisted research workflows
- dynamic curriculum orchestration for beta performance systems

Access will likely remain tied to Pro membership due to compute costs, aggregate infrastructure, advanced AI processing, and future offloading requirements.

The long-term vision:
> a continuously learning performance intelligence layer that helps users understand how training, recovery, nutrition, and supplementation interact together over time — both personally and collectively across the EVO ecosystem.

## Related
- [[EVO Architecture Bible]]
- [[EVOtraining — Adapter Behavior.md]]
- [[EVOtraining — Coach Application Philosophy.md]]
^[wiki-native — no upstream source]
