---
title: talent-training-natural-language
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/talent-training-natural-language.md
updated: 2026-07-24
---

# Talent Training Through Natural Language

One thing I'm starting to realize is that my original Talent training philosophy was too narrow.

Originally, the idea was simple:

The user performs a task.
Alice watches the task.
Alice proposes a Method.
If the Method is repeatedly approved, it eventually becomes a Talent.

And honestly, that still works really well for execution-heavy workflows.

Things like:
- file operations,
- terminal workflows,
- browser automation,
- repeatable UI interactions,
- tool orchestration.

But I'm starting to realize that some of the most important workflows in EVO are not really observable workflows.

They're reasoning workflows.

Things like:
- implementation planning,
- doctrine traversal,
- architecture analysis,
- governance reviews,
- issue hierarchy generation,
- comparing doctrine against the codebase,
- identifying reusable systems,
- identifying missing infrastructure.

Alice can't really "watch" me do those things.

She can watch terminal commands.
She can watch browser clicks.
She can watch tool usage.

But she cannot directly observe how I mentally connect architecture together.

She can't observe:
- how I compare doctrine to implementation,
- how I identify missing concepts,
- how I map systems together,
- or how I reason about execution order.

That creates a problem because some of the most valuable workflows in the system are reasoning-driven, not execution-driven.

So I think we need a second Talent training path.

Not just:
"watch me do this."

But also:
"let me explain how this workflow works."

The idea would be natural-language-guided Talent training.

Instead of requiring Alice to observe everything directly, the user can teach workflows conversationally.

For example, I could tell Alice:

"Traverse the canonical doctrine for Coach. Compare it against the existing EVOtraining codebase. Identify reusable runtime systems. Identify missing implementation areas. Generate an implementation plan. If you find ambiguity, ask me clarification questions."

At that point, I've effectively explained the workflow.

I've explained:
- the goal,
- the process,
- the boundaries,
- the expected outputs,
- and the clarification behavior.

Alice now has enough information to synthesize a Method.

And honestly, this starts looking more like a "Talent Training Talent."

Instead of hardcoding every workflow into the application, Alice learns how to construct reusable workflows from:
- doctrine,
- reasoning,
- clarification loops,
- user intent,
- and governance rules.

I also think there should be two approval modes.

Approve as Method.
Approve as Talent.

Approve as Method would be used when:
- the workflow is experimental,
- the workflow touches uncontrolled external systems,
- the workflow uses arbitrary web navigation,
- the workflow hasn't been proven safe yet,
- or the user has not demonstrated repeatability.

Methods remain reviewable and temporary.

Approve as Talent would only be allowed when:
- the workflow remains inside EVO-governed systems,
- the workflow uses trusted tools,
- the workflow remains deterministic enough,
- and the user has clearly defined the behavior.

So in this system, not every Method can become a Talent automatically.

There are governance boundaries.

For example:

If Alice has to randomly click around the web, that should never become an auto-approved Talent.

That remains a Method.

But if the workflow is entirely internal to EVO:
- doctrine traversal,
- repository analysis,
- issue generation,
- implementation planning,
- note classification,
- adapter analysis,
- cognition traversal,

then the risk is dramatically lower because the execution environment is already governed.

And honestly, this is especially important for Connect.

Because Connect is not really just an automation app.

It's an orchestration app.

A huge percentage of Connect workflows are reasoning-heavy workflows.

Which means observational learning alone will never be enough.

Another really important distinction is that Alice eventually has something Claude does not.

Claude only sees the context we inject.

Alice eventually has:
- doctrine memory,
- long-term cognition,
- workflow history,
- vector retrieval,
- prior clarification history,
- user-specific architectural memory.

So before Alice asks the user for clarification, she should first:
- search doctrine,
- search chat vectors,
- search prior implementation history,
- search previous workflow discussions,
- attempt ambiguity resolution internally.

Only after those fail should she escalate back to the user.

That is a major difference between:
"AI assistant planning"
and
"resident system cognition."

The real goal here is not simply:
"teach Alice how to repeat actions."

The real goal is:
"teach Alice how to construct safe reusable workflows from reasoning, doctrine, and user intent."

## Related

^[source-materials/mirrors/doctrine/talent-training-natural-language.md]
