
## Type
Concept

## Parent MOC
- [[MOC - EVOconnect (Modular OS Layer)]]

## Related Notes
- [[Connect - Hive Architecture]]
- [[Connect - Security & Privacy Model]]
- [[Connect - Delegator & Governance]]
- [[Connect - Task System]]
- [[Provider vs Environment Access]]

---

# Core Clarification

GU and GT are Training adapters.

However, the infrastructure used to create and distribute them should **not** be Training-specific.

Instead:

> **All global adapters should use one canonical federation creation and distribution pipeline**

That means this is an **EVOsystem architecture decision**, not a Connect-only decision and not a Training-only decision.

---

# Core Principle

## Domain-specific adapters
Examples:
- GU
- GT
- future domain-global adapters for Mind, Learn, Connect, etc.

These belong to their domains conceptually.

---

## Federation pipeline
Creation, storage, packaging, versioning, and distribution of global adapters should be:

- shared
- governed
- reusable
- domain-agnostic at the infrastructure level

That pipeline should not care whether the artifact is:
- GU
- GT
- future GM
- future GL
- future GC

It should only care that:
- a federation job exists
- an adapter artifact must be created
- the artifact must be versioned, stored, and distributed safely

---

# Correct System Framing

## Training owns
- GU / GT semantics
- how those adapters are interpreted in Training
- when Training consumes them
- training-specific adapter weighting / routing rules

## Shared federation system owns
- job orchestration
- aggregation workflow
- artifact generation pipeline
- artifact upload
- artifact version metadata
- distribution
- retention / replacement policy
- audit trail

---

# New Architectural Rule

> **Global adapter meaning is domain-owned**
> **Global adapter creation and distribution is system-owned**

This separation is the important part.

Without it:
- Training becomes the accidental owner of federation infrastructure
- future domains duplicate backend logic
- adapter distribution gets reimplemented per app
- global learning becomes fragmented

With it:
- Training provides one consumer case
- the system provides one reusable pipeline
- future domains can plug into the same flow

---

# Proposed Shared Architecture

## 1. Federation Job Layer
Shared system layer.

Responsibilities:
- create federation jobs
- define job type
- associate job with domain + adapter class
- track lifecycle
- store status and metadata

Example job types:
- global_user_adapter_build
- global_trainer_adapter_build
- future global_domain_adapter_build

---

## 2. Aggregation / Build Layer
Shared execution layer.

Responsibilities:
- gather eligible updates
- merge / aggregate inputs
- build final global adapter artifact
- validate artifact
- produce version metadata

This should be abstract enough that the pipeline does not care whether the adapter is GU, GT, or another future global adapter.

---

## 3. Artifact Storage Layer
Shared storage layer.

Responsibilities:
- upload generated adapter artifacts
- store versioned outputs
- expose artifact references
- support rollback / replacement

This is where R2 makes sense as the durable artifact plane.

---

## 4. Registry Layer
Shared metadata layer.

Responsibilities:
- track adapter version
- track domain ownership
- track build timestamp
- track compatibility
- track rollout state
- map active adapter → consuming domains

---

## 5. Distribution Layer
Shared delivery layer.

Responsibilities:
- make artifacts available to consuming runtimes
- define when clients pull new global adapters
- support staged rollout if needed
- support invalidation / revocation

---

# What This Means for Fly.io Replacement

The real replacement is not:

> “Move GU and GT off Fly.io”

The real replacement is:

> **Replace Fly.io with a shared federation backend path for all global adapters**

GU and GT are simply the first migration targets.

---

# Recommended Stack Framing

## Supabase should own
- job records
- orchestration state
- metadata
- permissions
- lifecycle tracking
- distribution registry

## R2 should own
- built adapter artifacts
- packaged federation outputs
- immutable or versioned output blobs

## Shared federation pipeline should own
- the creation workflow itself
- artifact validation
- publish / distribute transitions

---

# Important System Boundary

Do not define this as:

- Training federation backend
- Connect distribution backend
- app-specific adapter upload path

Define it as:

> **global adapter federation infrastructure**

Then attach domain-specific meaning on top.

---

# Suggested Canonical Terms

## Shared terms
- FederationJob
- GlobalAdapterBuild
- AdapterArtifact
- AdapterRegistryEntry
- DistributionRecord
- ActiveAdapterPointer

## Domain terms
- GU
- GT
- future domain-global adapter names

This keeps infrastructure neutral and domain semantics expressive.

---

# Why This Is the Right Move

Because the moment Mind, Learn, or Connect needs its own global adapter class, you do **not** want to invent a second backend pattern.

You want:

- one build pipeline
- one artifact pipeline
- one registry model
- one distribution model

That is what makes the system scalable.

---

# Recommended Design Rule

## Shared federation system must be parameterized by:
- domain
- adapter type
- build strategy
- output format
- rollout strategy

## Shared federation system must NOT be hardcoded to:
- Training
- GU / GT naming
- trainer-specific assumptions
- workout-specific semantics

---

# Clean System Statement

> **Training defines GU and GT**
> **EVOsystem defines how any global adapter is built and distributed**

That is the clean separation.

---

# Migration Interpretation

## Immediate practical meaning
When replacing Fly.io:

- do not create a Training-only replacement
- create the first version of the shared federation creation/distribution path
- migrate GU and GT onto it first

## Later
- add more global adapter classes without changing the infrastructure model

---

# Core Takeaway

This is not a Connect architecture decision.

This is an **EVOsystem federation infrastructure decision**.

GU and GT are the first concrete artifacts,
but the backend path should be the canonical path for:

> **all global adapter creation and distribution going forward**