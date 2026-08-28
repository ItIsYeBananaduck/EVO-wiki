---
title: "EVOconnect — Action Bar & Mini Action Bar System (Raw Draft)"
type: doctrine
tags: ['lsctech', 'doctrine', 'source-material', 'evo']
updated: 2026-08-19
---

# EVOconnect — Action Bar & Mini Action Bar System (Raw Draft)

## Purpose

The Action Bar and Mini Action Bar system defines how Connect exposes workflows, lightweight interactions, operational visibility, and temporary pane access without forcing the user to keep large workspace surfaces open.

The goal is to reduce workspace clutter while preserving workflow continuity and contextual awareness.

The Action Bar is not intended to function as a simple launcher.

It is intended to function as:
- a workspace orchestration layer
- a workflow visibility layer
- a lightweight operational monitor
- a temporary pane system
- an approval surface
- a persistent continuity layer

Mini Action Bars extend this concept to collapsed panes.

---

# Core Principle

The Action Bar exists to keep workflows accessible without keeping the entire workspace visible.

The user should always feel connected to:
- current workflows
- current Alice activity
- approval requirements
- lightweight interactions
- temporary workspace access

without requiring the full Connect workspace to remain expanded.

---

# Action Bar Philosophy

Traditional docks and launchers mostly expose applications.

The Connect Action Bar exposes:
- workflows
- panes
- workspace state
- operational activity
- contextual interactions
- lightweight notifications

The Action Bar should feel alive and workflow-aware rather than static.

---

# Relationship to Modes

The Action Bar behaves differently depending on the current Connect mode.

## Pane Mode

In Pane Mode:
- the Action Bar remains lightweight
- mini action bars become more important
- quick interactions are prioritized
- collapsed panes remain highly accessible

## Command Center Mode

In Command Center:
- the Action Bar becomes the primary orchestration layer
- views may restore from the Action Bar
- temporary panes may launch from the Action Bar
- workflow state may surface through the Action Bar
- approvals and operational visibility become more important

## HUD Mode

In HUD Mode:
- the HUD acts as the lightweight persistent access layer
- entering Command Center expands the Action Bar
- leaving Command Center may collapse workflows back into Action Bar visibility

---

# Action Bar Responsibilities

The Action Bar is responsible for:
- launching panes
- restoring panes
- restoring views
- launching temporary panes
- exposing workflow state
- exposing lightweight notifications
- surfacing approvals
- reducing workspace clutter
- preserving operational continuity

The Action Bar should remain persistent even when larger workflows collapse or hibernate.

---

# Temporary Pane System

One of the Action Bar’s most important responsibilities is temporary pane orchestration.

Temporary panes allow users to:
- quickly access functionality
- perform lightweight work
- collapse the pane afterward
- continue their main workflow uninterrupted

Examples:
- temporary EVOterminal access
- temporary notes access
- temporary Git access
- temporary browser access
- temporary Connect Library access

Temporary panes prevent users from needing to permanently dedicate workspace space to every tool.

---

# Workflow Visibility

The Action Bar should expose lightweight operational visibility.

Examples:
- current Alice task
- current terminal command
- last command output line
- workflow progress
- waiting state
- blocked state
- completion state
- approval required state

This allows users to stay aware of ongoing activity without opening large panes.

---

# Approval Surface

The Action Bar should act as a lightweight approval surface.

When Alice requires user input, approval, or intervention, the Action Bar may surface:
- approval requests
- warnings
- blocked workflows
- resource recommendations
- confirmation actions

Examples:
- close third-party app?
- run privileged command?
- hibernate workflow?
- continue automation?

This allows approvals to remain lightweight and contextual.

---

# Mini Action Bars

Mini Action Bars are lightweight contextual controls attached to collapsed panes.

A Mini Action Bar allows a collapsed pane to remain useful without reopening the full pane.

Mini Action Bars may expose:
- quick actions
- lightweight notifications
- workflow status
- current task state
- approval indicators
- active Alice activity

The goal is to compress workflows visually without destroying workflow continuity.

---

# Quick Interactions

Mini Action Bars should support lightweight contextual interactions.

Examples:

Task pane:
- quick add task
- mark complete
- next task
- reopen active task

Notes pane:
- quick scratch
- paste to note
- quick capture
- reopen recent note

Terminal pane:
- rerun command
- inject snippet
- reopen terminal session
- stop active process

Browser pane:
- save page
- summarize page
- quick clip

The goal is to minimize workflow interruption.

---

# Collapsed But Live Behavior

Collapsed panes do not necessarily become inactive.

A collapsed pane may remain:
- lightly active
- operationally aware
- monitoring workflows
- exposing state
- waiting for approvals

This is especially important for:
- Task Manager
- EVOterminal
- Alice workflows
- lightweight monitoring

Collapsed workflows are intended to feel compressed rather than destroyed.

---

# Action Bar Clutter Reduction

The Action Bar should dynamically reduce clutter.

Examples:
- pinned panes may disappear from the Action Bar
- inactive workflows may collapse
- grouped panes may consolidate controls
- temporary panes may disappear after use
- inactive workflow indicators may minimize automatically

The Action Bar should feel adaptive and operationally aware.

---

# Relationship to Resource Management

The Action Bar becomes more important during resource-saving behavior.

When workflows collapse, suspend, or hibernate:
- the Action Bar remains available
- lightweight workflow visibility remains available
- approval requests remain visible
- workflows remain restorable

This allows Connect to preserve operational continuity while reducing workspace and resource usage.

---

# Alice’s Role

Alice uses the Action Bar as a lightweight communication and orchestration surface.

Alice may:
- expose workflow state
- surface approvals
- expose lightweight notifications
- communicate resource pressure
- recommend suspensions
- recommend hibernation
- surface workflow completion

The Action Bar allows Alice to remain operationally present without requiring constant full workspace interaction.

---

# Long-Term Direction

The long-term goal of the Action Bar system is to make Connect feel like:
- a living operational workspace
- a persistent workflow layer
- an adaptive orchestration system
- a lightweight AI-aware desktop layer

rather than:
- a static launcher
- a traditional dock
- a collection of disconnected windows

The Action Bar should ultimately become the user’s primary lightweight connection point to their workflows, panes, and Alice activity.
