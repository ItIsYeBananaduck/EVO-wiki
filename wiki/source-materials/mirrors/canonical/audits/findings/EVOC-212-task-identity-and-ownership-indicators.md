---
type: audit-finding
---

> **Status: Implementation Artifact**
> EVOfy/openclaw architecture and governance design note. Active design contract for EVOconnect/EVOfy execution backbone. See evofy/README.md for context.

# EVOC-212 — Task Identity and Ownership Indicators

> Status: Draft contract (architecture only; no UI implementation).
> Date: 2026-04-02.
> Parent: EVOC-210 — Connect Core Interaction Contract.

## 1) Purpose

Define the contract for how timeline dots communicate **task identity** and **ownership** without overloading color with execution state semantics.

This contract covers:

- strict separation of identity semantics vs state semantics,
- controlled (bounded) identity color palette,
- ownership markers inside timeline dots,
- and compatibility with EVOC-213/EVOC-215 timeline rules.

## 2) Core invariants

1. Dot color represents **task identity**, not task state.
2. State MUST NOT be encoded by color, hue shifts, gradients, or color-only overlays.
3. Identity colors are selected from a **controlled palette** (finite token set), not an infinite/generated gradient.
4. Task ownership must be represented inside the dot via either:
   - an owner icon, or
   - a short owner letter marker.
5. State remains inferable from existing timeline channels only:
   - position,
   - fill,
   - animation.

## 3) Identity color contract

### 3.1 Palette constraints

- Identity color assignment MUST use a predefined token list (for example: `identity.1` ... `identity.N`).
- `N` must be finite and versioned by design-system contract.
- Runtime-generated unconstrained gradients are prohibited for task identity.
- Palette updates require an explicit contract/version change and migration guidance.

### 3.2 Stability requirements

- A task's identity color MUST remain stable for the life of that task within an execution stream.
- Re-rendering, pagination, or cluster expansion/collapse must not remap identity color for the same task.
- If palette exhaustion occurs (more distinct identities than tokens), collision handling must follow deterministic fallback (see Section 6) while preserving non-color ownership markers.

## 4) Ownership indicator contract

Each task dot MUST include an ownership marker, selected by capability and density constraints:

1. **Preferred:** owner icon (agent/system/user glyph) when legible at current density.
2. **Fallback:** single-letter owner marker inside the dot when icon is unavailable or illegible.
3. Marker content must remain deterministic per owner identity.
4. Marker is identity metadata and MUST NOT mutate with task state transitions.

Rules:

- Ownership marker must be visible in normal density mode.
- In high-density cluster mode, marker may be abbreviated but task discoverability must remain available after expand interaction (EVOC-215).
- Marker changes are allowed only when ownership actually changes in the execution model (e.g., reassignment event).

## 5) State semantics separation

To preserve EVOC-213 task visibility semantics, state must remain encoded only via:

- **position** = task order,
- **fill** = progress/completion amount,
- **animation** = active execution.

Prohibited patterns:

- using red/green/amber to indicate task state,
- dimming/saturating identity color to represent blocked/failed/completed,
- using palette hue ramps as pseudo-progress indicators.

If additional state emphasis is needed, use non-color channels already allowed by timeline contracts (fill mode, motion, stroke pattern, or position context), without reassigning color meaning.

## 6) Collision and accessibility behavior

When multiple tasks resolve to the same identity color token:

1. preserve assigned color token,
2. distinguish tasks through ownership marker and existing timeline semantics,
3. if needed, add non-color differentiators (outline style/offset) per EVOC-215 distinguishability rules.

Accessibility guardrails:

- Dot meaning must be understandable without color perception.
- Ownership indicator (icon/letter) and state channels (position/fill/animation) must remain sufficient for interpretation.
- Contrast and legibility thresholds are design-system concerns and must be validated at implementation time.

## 7) Event contract (identity/ownership)

Required event types:

- `task_identity.identity_assigned`
- `task_identity.identity_remapped` (rare; only on explicit contract/version migration)
- `task_identity.ownership_assigned`
- `task_identity.ownership_changed`

Minimum fields:

- `executionId`
- `taskId`
- `identityColorToken`
- `identityPaletteId` (string)
- `identityPaletteVersion` (semver/string)
- `ownerId`
- `ownerMarkerType` (`icon` | `letter`)
- `ownerMarkerValue` (nullable for icon-only systems)
- `timestamp`
- `surface`

Constraints:

- `identityColorToken` must reference a known controlled-palette token valid within the specified `identityPaletteId` and `identityPaletteVersion`.
- `identityPaletteId` and `identityPaletteVersion` are required to enable deterministic replay of identity assignments across palette migrations.
- `ownerMarkerType` and `ownerMarkerValue` must map deterministically to owner identity independently of `identityPaletteVersion`. (Owner marker mapping is orthogonal to identity color palette versioning; owner markers represent task ownership, not task identity color semantics.)
- State transitions (`queued`, `active`, `completed`, `blocked`, `failed`) MUST NOT require identity reassignment.
- Consumers MUST use `identityPaletteId` and `identityPaletteVersion` when replaying events or auditing identity assignments to ensure correct token resolution.

Event schema requirements:

- `task_identity.identity_assigned` and `task_identity.identity_remapped` MUST include `identityPaletteId` and `identityPaletteVersion` fields.
- These fields enable consumers to deterministically replay identity assignments even when palette definitions evolve.

## 8) Acceptance criteria

This issue is complete when:

1. Color semantics are explicitly defined as **identity-only**.
2. Controlled palette requirement is explicit and infinite gradients are disallowed.
3. Ownership indicator requirement (icon or letter inside dot) is explicit.
4. Explicit prohibition exists for color-as-state encoding.
5. State channels are constrained to position/fill/animation and aligned with EVOC-213.
6. Event payload minimums for identity/ownership assignment are documented.

## 9) Out of scope

- visual theme styling and exact token values,
- platform-specific rendering details,
- animation choreography/polish,
- final iconography art direction.