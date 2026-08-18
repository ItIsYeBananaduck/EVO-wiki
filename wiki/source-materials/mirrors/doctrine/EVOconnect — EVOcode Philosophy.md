# EVOconnect — EVOcode Philosophy (Raw Draft)

## Purpose

EVOcode defines how coding workflows fit inside Connect.

EVOcode is not intended to be a monolithic IDE replacement.

EVOcode is a code-aware pane that participates in the larger Connect workspace.

The goal is not to build another VS Code clone. The goal is to let users assemble their own development environment from modular Connect panes.

In this model, EVOcode provides the code editing surface, but Connect becomes the IDE through pane composition.

---

## Core Principle

EVOcode is a pane, not the IDE.

The IDE is the view the user creates by combining panes.

A development view may include:
- EVOcode
- EVOterminal
- Git
- Alice Chat
- Tasks
- Living Notes
- Scratches
- EVObrowser
- Connect Library

Together, these panes create the development environment.

This allows the user to build:
- a lightweight coding workspace
- an advanced IDE-style workspace
- a debugging workspace
- a research and coding workspace
- a project-management-driven coding workspace

without requiring one giant permanently active IDE process.

---

## EVOcode as a Code-Aware Text Pane

Architecturally, EVOcode is closer to Living Notes and Scratches than to a traditional IDE.

Living Notes and Scratches are primarily markdown-oriented panes.

EVOcode is similar, but code-oriented.

The main differences are:
- syntax highlighting
- code formatting
- file awareness
- repo awareness
- code navigation
- developer-focused editing behavior

At its core, EVOcode is a large text editor pane that understands code formatting and development context.

That simplicity is intentional.

---

## Connect Becomes the IDE

Connect should not attempt to recreate every IDE feature inside one pane.

Instead, Connect creates IDE behavior by combining independent panes.

For example:

A simple coding view may use:
- EVOcode in the center
- Alice Chat on the right
- EVOterminal at the bottom
- Tasks on the left

A more advanced coding view may use:
- EVOcode in the center
- Alice Chat and Notes on the right
- Git and Tasks on the left
- EVOterminal at the bottom
- EVObrowser as a temporary pane from the Action Bar

This allows users to create the IDE that fits their workflow instead of forcing every user into the same layout.

---

## Manual Mode vs Alice Mode

EVOcode should distinguish between manual user coding and Alice-driven coding.

### Manual Mode

Manual Mode optimizes for the human.

When the user is actively coding, Connect may keep user-facing coding helpers active.

Examples:
- syntax highlighting
- autocomplete
- formatting helpers
- linting
- previews
- language-specific extensions
- Git visibility

The goal is to make manual development comfortable.

### Alice Mode

Alice Mode optimizes for execution.

When Alice is coding for the user, she usually does not need the full human-facing editor stack.

Alice primarily needs:
- repo access
- file read/write access
- terminal access
- Git access
- MCP tools
- task context
- notes context
- Delegator approval flow

In Alice Mode, heavy user-facing extensions can be suspended unless they are needed for the workflow.

This keeps Connect lighter while Alice works.

---

## MCP and Tools vs Extensions

MCP tools and extensions serve different purposes.

MCP tools are primarily for Alice execution.

Extensions are primarily for the user experience.

Alice does not need every visual or convenience feature a human developer needs.

Alice needs capabilities:
- read files
- search repo
- edit files
- run commands
- inspect output
- run tests
- inspect Git state
- update tasks
- create summaries

The user may need conveniences:
- syntax highlighting
- autocomplete
- language helpers
- previews
- file explorers
- visual Git tools
- debugger panels

This distinction keeps EVOcode from becoming bloated.

---

## Extension Activation

Extensions should be optional and context-aware.

If a Python extension is installed but the user is not working in Python, it should not need to stay active.

If a Flutter extension is installed but the current workflow does not involve Flutter, it can sleep.

Alice may help manage extension state based on:
- active files
- active repo
- current task
- current view
- user mode
- available resources

The goal is to activate only what is useful for the current workflow.

---

## Resource Philosophy

EVOcode should avoid becoming a resource-heavy permanent process.

Traditional IDEs often consume large amounts of memory because many tools, extensions, language servers, watchers, and background services remain active even when they are not needed.

Connect should treat those pieces as modular and conditional.

When they are useful, they can run.

When they are not useful, they can sleep.

This keeps development workflows powerful without forcing the user to pay the resource cost all the time.

---

## Alice’s Role in Coding Workflows

Alice should not be trapped inside EVOcode.

Alice exists across the whole Connect workspace.

In coding workflows, Alice may use:
- EVOterminal
- Git
- Tasks
- Living Notes
- Scratches
- EVObrowser
- MCP tools
- Connect Library
- EVOcode when it is open

If EVOcode is not open, Alice can still perform many coding workflows through the terminal, file access, Git, and MCP tools.

If EVOcode is open, Alice can use it as a visualization and collaboration surface.

This means EVOcode is useful for human review and manual intervention, but it is not required for Alice to work.

---

## User Visibility

EVOcode gives users a way to see and participate in what Alice is doing.

When Alice edits code, EVOcode may show:
- changed files
- diffs
- highlighted edits
- explanations
- related tasks
- terminal output
- test results
- linked notes

The user should be able to move between:
- watching Alice work
- reviewing Alice’s work
- manually editing
- taking over the workflow
- asking Alice for clarification

This creates a collaborative development environment rather than a black-box agent.

---

## Smart Comments

Smart comments are a long-term idea for making codebases more understandable.

A smart comment is an interactive comment or marker that allows the user to open a focused Alice chat about a file, function, system, or decision.

A smart comment may help explain:
- what a file does
- why a function exists
- how a system connects to other systems
- what architectural decision led to the code
- what risks or dependencies exist
- what task or note is related to the code

Smart comments turn the codebase into an interactive learning surface.

The goal is not to fill the codebase with noisy comments.

The goal is to create contextual entry points where the user can ask Alice for focused explanations.

---

## Educational Coding Layer

EVOcode can become a learning environment over time.

This matters because not every user will be an expert developer.

A user may want to understand:
- what Alice changed
- why a file matters
- how a function works
- what a command did
- what a test failure means
- how a feature connects to the product

EVOcode should allow Alice to teach through context.

This turns coding workflows into learning workflows without requiring a separate education app.

---

## Development Views

Development workflows should be built as views.

Examples:
- lightweight coding view
- debugging view
- PR review view
- research and implementation view
- task-driven coding view
- learning-oriented coding view

Each view may define:
- visible panes
- temporary panes
- grouped panes
- Action Bar shortcuts
- extension behavior
- resource behavior
- Alice context

This allows different coding workflows to have different workspace shapes.

---

## Long-Term Direction

The long-term goal of EVOcode is to give users the power of an IDE without forcing Connect to become a monolithic IDE.

EVOcode should remain:
- modular
- code-aware
- resource-conscious
- pane-based
- Alice-integrated
- user-configurable

Connect becomes powerful because the user can assemble the workflow they need.

EVOcode is only one pane in that system.

The larger idea is:

The user does not open an IDE.

The user builds their development environment inside Connect.
