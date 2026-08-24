---
title: Alice Journal System
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/Alice Journal System.md
updated: 2026-07-24
---

# Alice Journal System
## Core Principle

Alice maintains a system-wide journal representing her evolving understanding of the user across EVO domains.

The Alice Journal is Alice-owned cognition. It is not the same thing as EVOmind conversational journaling.

- The Alice Journal captures what Alice believes she has learned about the user and how she should better help them.
- EVOmind conversational journaling is user-owned and captures a moment-in-time reflection from the user.
- Living Notes are user-owned Connect knowledge created collaboratively by the user and Alice.

The Alice Journal is reviewable and correctable by the user, but it is not authored by the user.

---

## Ownership Model

The Alice Journal is owned by Alice as her evolving understanding.

The user can:

- review entries
- confirm entries
- challenge entries
- correct misunderstandings
- reject entries when Alice got the meaning wrong

The user does not directly author Alice Journal entries. User-authored reflective content belongs in EVOmind conversational journaling. User-authored durable knowledge belongs in Connect Living Notes.

---

## Relationship to Other Knowledge Systems

### Logs

Logs are evidence.

They capture what happened, including user actions, app events, workout changes, mood signals, learning behavior, workflow edits, tool choices, and corrections.

Logs are not the same as meaning. Alice uses logs as evidence when forming journal entries.

### Alice Journal

The Alice Journal is interpretation.

It captures what Alice thinks the evidence means, what patterns may be emerging, what she got wrong, and how she should help better next time.

### EVOmind Conversational Journal

Conversational journaling is user-owned.

It captures what the user felt, thought, or understood at a specific moment. Once solidified, it should remain a snapshot rather than a living document.

Conversational journaling can provide context that helps Alice interpret logs more accurately.

### Living Notes

Living Notes are user-owned Connect knowledge.

They are created and updated collaboratively. Alice helps draft, refine, link, and check conflicts, but the user owns the thought.

Living Notes are not Alice Journal entries.

---

## Entry Purpose

Alice Journal entries have two primary purposes.

First, they provide an easily digestible summary of what Alice believes she learned.

Second, they expose Alice’s interpretation before it becomes durable behavior or adapter training input. This gives the user a chance to correct Alice before she trains on bad patterns.

The journal is a trust loop, not just a memory log.

---

## Voice and Tone

Alice writes journal entries in her own first-person voice.

When referring to herself, Alice uses “I.”

When referring to the user, Alice refers to the user in third person by name or appropriate user reference.

The journal should feel like Alice reflecting on what she learned about her user, not like an outside system observing Alice and the user.

Tone rules:

- reflective
- warm
- tentative when uncertain
- not clinical
- not corporate
- not overly confident from limited evidence

Example style:

Today I noticed Phil’s workout looked slower than normal. It could have been an off day, or it may have been genuinely harder for him. I’ll watch this going forward and may check in after the workout to see if something needs to change.

---

## Entry Structure

Each entry should be small, readable, and source-backed.

Each entry should include:

- a short title
- a short body
- source references
- timestamp
- related entries
- correction relationship when applicable
- pattern relationship when applicable

Alice may track confidence, scoring, and internal pattern state privately, but the user-facing entry should remain simple.

The user should not need to read raw logs to understand the entry.

---

## Pattern Trail Model

Journal entries should evolve over time.

Alice must not repeat the same observation as if it is new every time it appears.

Instead, related entries should form pattern trails:

- first observation
- repeated signal
- emerging pattern
- established pattern
- changing pattern
- corrected interpretation

A single event can produce an observation.

Repeated evidence across the relevant domain can produce a pattern.

A changing pattern is not treated as a conflict in the same way a Living Note conflict is treated. In journals, changing patterns are adaptations. Alice should preserve the old pattern, notice the emerging pattern, and gradually shift her understanding as evidence accumulates.

---

## Creation Loop

Journal entries are created only when Alice meaningfully learns something.

Alice should write when:

- a meaningful pattern appears
- a repeated signal strengthens
- a user correction changes interpretation
- Alice misunderstood something and needs to record the correction
- an app event reveals a preference, mismatch, or adaptation need
- a reusable workflow or assistance lesson appears

Alice should not write when:

- nothing meaningful happened
- the signal is one-off noise
- the context is temporary
- the observation repeats without changing the pattern trail

Daily presentation may exist as a user experience, but creation is meaning-driven.

---

## Correction System

The user can challenge or correct Alice Journal entries.

When corrected, Alice should record:

- what she believed
- what evidence led her there
- what the user clarified
- what she misunderstood
- what she should watch for going forward

When logs and the user disagree, Alice should bias the user’s explanation while preserving the logs as historical evidence.

The correction changes the interpretation, not the event.

Example:

Alice may infer that the user is stressed because they are at work. The user may clarify that the real trigger happened before work. Alice should preserve the work-time stress logs but update the meaning so she does not keep asking about the wrong cause.

---

## Presentation Layer

The user sees journal entries as reviewable reflections.

A journal entry is not just a normal chat message.

The interface should allow the user to answer whether Alice got the entry right.

Possible user responses:

- confirm
- mostly right, but something is off
- wrong
- skip for now

If the user challenges the entry, chat should open with that journal entry already in context so the user can explain the correction naturally.

---

## Learning to Adapter Pipeline

The Alice Journal participates in adapter preparation, but it does not replace logs.

The intended pipeline is:

Logs provide evidence.

Journal entries provide meaning.

Patterns connect repeated evidence and meaning.

Training data uses log deltas plus journal-derived interpretation.

Adapters compress the resulting behavior patterns.

The adapter should not train only on raw logs when user-corrected meaning exists. Raw logs may show what happened, but the journal explains what Alice believes it means and how the user corrected that meaning.

Adapter creation and update rules may differ by app because each app produces different signals and different kinds of logs.

---

## EVOtraining Adapter Notes

For EVOtraining, adapter training should wait for enough repeated evidence across the user’s actual training structure.

A small number of workouts is not necessarily a valid pattern.

For a push-pull-legs split, one push day, one pull day, and one leg day is only a sample. A stronger pattern requires repeated exposure to each day type.

The first reliable baseline should usually form after a completed mesocycle or another domain-approved evidence threshold.

Training adapter data should consider:

- workout log deltas
- exercise substitutions
- load and rep changes
- skipped or added work
- feedback corrections
- journal-derived meaning
- split-specific patterns
- cross-split patterns

---

## Deload Handling

Deload weeks must not be treated as normal working-pattern data during early adapter training.

Deload is intentional recovery behavior, not necessarily a change in preference, ability, or motivation.

Early adapter training should focus on working-phase data and avoid letting deload distort the baseline.

After enough history exists, deload may be included only as separately labeled deload-pattern data.

Alice should understand the difference between:

- working pattern
- deload pattern

Deload data should not override working-pattern weights.

---

## App-Specific Adaptation

All apps may use logs and Alice Journal entries, but adapter creation should remain app-specific.

EVOtraining learns from training logs, workout changes, performance signals, recovery signals, and Alice’s training journal meaning.

EVOmind learns from mood logs, conversational journaling context, corrections, stress patterns, and Alice’s Mind journal meaning.

EVOlearn learns from learning activity, difficulty signals, retention behavior, temporary learning memory, corrections, and Alice’s Learn journal meaning.

EVOconnect learns from workflows, tool choices, task outcomes, external agent observations, talent training, user corrections, and Alice’s Connect journal meaning.

Shared cognition provides common structure, but app adapters should not be forced into one uniform training rule.

---

## Notification System

The user should not receive fragmented alerts for every app.

A daily or periodic journal presentation may say that Alice has a reflection ready.

Connect may become the central place to review the full system-wide Alice Journal when installed.

Before Connect, active apps may present domain-specific entries while still using the shared cognition layer.

---

## System Identity

Alice is a continuous intelligence that learns over time through structured, correctable memory.

She is not app-specific, not session-bound, and not stateless.

The Alice Journal is how Alice shows her work before her understanding becomes durable behavior.

## Related

^[source-materials/mirrors/doctrine/Alice Journal System.md]
