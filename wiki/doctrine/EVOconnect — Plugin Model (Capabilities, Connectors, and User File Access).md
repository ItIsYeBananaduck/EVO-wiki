---
title: EVOconnect — Plugin Model (Capabilities, Connectors, and User File Access)
type: concept
tags: ["EVO","doctrine"]
sources:
  - source-materials/mirrors/doctrine/EVOconnect — Plugin Model (Capabilities, Connectors, and User File Access).md
updated: 2026-07-24
---

# EVOconnect — Plugin Model (Capabilities, Connectors, and User File Access)
#connect ## Core Idea
Plugins are how Alice gains bounded access to external systems.
A plugin is not just “an integration.”
In EVOconnect, a plugin is a governed capability layer that allows Alice to: - read or write data in an external system - perform approved actions in that system - expose those actions as Methods and Talents - use important user files as part of task execution
Plugins must make external systems feel native to Connect without exposing raw complexity to the user.

What a Plugin Is
A plugin should provide three things:
1. Connection
A way to securely connect to an external app, service, or local system.
Examples: - QuickBooks - Excel - Google Drive - local folders - email - calendar - CRM tools - project tools

2. Capabilities
A defined list of things Alice is allowed to do.
Examples: - create invoice - fetch client list - read spreadsheet - update row - send email - find file - create calendar event
A plugin should never expose “everything.” It should expose: - specific actions - scoped access - bounded operations

3. Resources
A structured way to surface important user data.
Examples: - files - folders - spreadsheets - reports - records - templates - recent documents
This lets Connect build a user-facing concept like:
“Important Files” or “Business Resources” or “Things Alice Can Use”
without forcing the user to think in terms of app boundaries.

What a Plugin Is Not
A plugin is not:
an unrestricted bridge into another app
a raw API wrapper exposed directly to the user
a hidden automation engine with no visibility
a bypass around Delegator
Plugins do not exist to give Alice unlimited power.
They exist to give Alice safe, explainable, useful capability.

The Connect View of Plugins
The user should not think:
“I am using the QuickBooks plugin”
“I am opening the Excel integration”
The user should think:
“Alice can handle my invoices”
“Alice can find my spreadsheet”
“Alice can pull the numbers I need”
So plugins are implementation layers, not primary UX concepts.

Plugin Types
Connect will likely need more than one kind of plugin.

1. App Plugins
These connect to external software platforms.
Examples: - QuickBooks - Notion - Google Drive - Dropbox - Slack - Excel / Microsoft 365 - Gmail - Google Calendar
Purpose
Allow Alice to: - perform actions - read structured data - update records - complete workflows

2. File Plugins
These expose important files and folders to Connect.
Examples: - local folders - synced cloud folders - external drives - document libraries
Purpose
Allow Alice to: - find user-important files - reference them in tasks - organize them into a meaningful directory - use them as resources for Methods and Talents
This is important because many users do not think: - app first
They think: - file first

3. Device / System Plugins
These expose device-level capabilities.
Examples: - camera - microphone - file picker - local notifications - printing - contacts - share sheet - clipboard
Purpose
Allow Alice to: - interact with the local environment - complete tasks on-device - bridge between apps and files

4. Connect Internal Plugins
These are capabilities native to the EVO ecosystem.
Examples: - task manager - talent registry - hive controls - method runner - delegation controls
Purpose
Make Connect itself extensible and modular.

Plugin Architecture Principle
A plugin should expose capabilities, not complexity.
Each plugin should define:
what it connects to
what Alice can do with it
what data it can access
what permissions are required
what Methods can be built from it
what Talents can be supported

Minimum Plugin Structure
Every plugin should have something like:
Identity
plugin id
name
description
icon
category
Permissions
what access it needs
why it needs it
whether access is read-only or read/write
Capabilities
list of allowed actions
input/output requirements
constraints
Resources
files, entities, records, or items it exposes
Method Support
what kinds of Methods can be built from it
Talent Support
what system Talents ship with it

Example — QuickBooks Plugin
Connection
User links QuickBooks account.
Permissions
read invoices
create invoices
read customers
update invoices
Capabilities
create invoice
send invoice
fetch invoice
list customers
get revenue report
Resources
invoices
customers
reports
templates
System Talents
Create invoice
Send invoice
Find overdue invoices
Generate basic report

Example — Excel / File-Oriented Plugin
Connection
User links local folder, OneDrive, or Excel source.
Permissions
read spreadsheet files
update specific sheets
create workbook copy
Capabilities
open workbook
read sheet
update row
filter table
export report
Resources
saved workbooks
frequently used files
budget sheets
planning templates
System Talents
Update weekly budget
Pull sales numbers
Find latest spreadsheet
Generate report from workbook

Important User Files Directory
This part matters a lot.
Connect should have a concept like:
Important Files
or
Alice Resources
This is not just an OS file browser.
It is a curated layer of: - frequently used documents - business-critical files - personal reference files - plugin-exposed documents - file shortcuts - structured resources from external apps
Why this matters
Users often care about: - “my spreadsheet” - “that invoice file” - “my payroll document” - “the report I use every week”
not: - where exactly it lives technically
So Connect should unify: - local files - cloud files - plugin documents - structured app resources
into one resource model Alice can work with.

How Alice Should Use Plugins
Alice should never just “have a plugin.”
She should:
know the plugin exists
know what it can do
know whether permission is granted
propose a Method using it
ask for approval if needed
execute through Delegator
log the action
So plugin usage always stays: - explicit - governed - auditable

Relationship to Talents
Plugins enable Talents.
Plugin gives:
capability
Method gives:
procedure
Talent gives:
reusable trusted outcome
This means:
A plugin without Talents is not user-ready.
At minimum, each plugin should ship with a starter set of system Talents.

Relationship to Delegator
Delegator must sit above plugins.
That means a plugin cannot: - run arbitrary actions - exceed its declared scope - access unrelated data - silently execute privileged workflows
Every plugin action must remain: - bounded - inspectable - controllable

Recommended Mental Model
Think of a plugin as:
a driver + capability manifest + resource adapter
It is the layer that translates an outside system into something Alice can safely understand and use.

Suggested First Implementation Strategy
Do not start with a giant universal plugin system.
Start smaller:
Phase 1
define one plugin contract
build 1–2 simple plugins
expose a few capabilities
test resource handling
test Method creation
test Delegator boundaries
Good starter plugins
local files / folder resources
Google Drive or Dropbox
calendar
QuickBooks later once the model is solid

Core Takeaway
Plugins are how Alice gains safe access to outside systems.
They should make external apps, files, and tools feel like native capabilities inside Connect.
The user should not have to learn the external system.
They should only need to express intent, review the proposed method, and get the outcome.

## Related

^[source-materials/mirrors/doctrine/EVOconnect — Plugin Model (Capabilities, Connectors, and User File Access).md]
