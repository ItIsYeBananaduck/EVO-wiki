---
type: audit-finding
---

> **Archived — Promoted to Lifecycle System**
> - **Lifecycle stage**: spec
> - **Domain**: connect
> - **Archival date**: 2026-05-12
> - **Archival reason**: Raw note classified and promoted to EVOnotes lifecycle system.
> - **Note**: Canonical/reference copy lives in docs/EVOnotes/spec/connect/. This file is intake history only.

> **Status: Implementation Artifact**
> Canonical `journal_entry` contract for EVO Cognition Layer with mapper responsibilities per domain. Active schema contract.

# EVOS1-269 — Cross-App `journal_entry` Contract and App Mappers

## Purpose

Define a canonical, implementation-ready `journal_entry` contract for the EVO Cognition Layer, and define mapper responsibilities for EVOtraining, EVOmind, EVOlearn, and EVOconnect.

**Journal definition**:
> "What did Alice learn about the user today, and how can she better help them in this domain?"

---

## Doctrine Alignment (EVO Cognition Layer)

This contract belongs to the **Cognition Layer (learning layer)**, not the event ingestion layer and not the Living Notes system.

- **Journals** = synthesized, domain-scoped learning artifacts produced from app events + interactions.
- **Logs** = raw/near-raw event and interaction records.
- **Living Notes** = user-facing long-lived note artifacts with separate lifecycle, storage, and editing semantics.

### Hard Boundary: Journals are NOT Living Notes

`journal_entry` objects MUST NOT be persisted in Living Notes tables/collections, and Living Notes artifacts MUST NOT be treated as canonical journal records.

---

## Canonical `journal_entry` Contract

### Field Schema (JSON)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "evo://contracts/cognition/journal_entry.schema.json",
  "title": "journal_entry",
  "type": "object",
  "additionalProperties": false,
  "required": [
    "entry_id",
    "user_id",
    "app_domain",
    "created_at",
    "effective_date",
    "summary",
    "learned_about_user",
    "help_strategy",
    "confidence",
    "privacy_tier",
    "visibility",
    "source_references",
    "corrections",
    "flags",
    "version"
  ],
  "properties": {
    "entry_id": { "type": "string", "minLength": 1 },
    "user_id": { "type": "string", "minLength": 1 },
    "app_domain": {
      "type": "string",
      "enum": ["EVOtraining", "EVOmind", "EVOlearn", "EVOconnect"]
    },
    "created_at": { "type": "string", "format": "date-time" },
    "effective_date": { "type": "string", "format": "date" },
    "summary": { "type": "string", "minLength": 1 },
    "learned_about_user": {
      "type": "array",
      "minItems": 1,
      "items": { "type": "string", "minLength": 1 }
    },
    "help_strategy": {
      "type": "array",
      "minItems": 1,
      "items": { "type": "string", "minLength": 1 }
    },
    "confidence": { "type": "number", "minimum": 0, "maximum": 1 },
    "privacy_tier": {
      "type": "string",
      "enum": ["pt0_public", "pt1_user", "pt2_sensitive", "pt3_restricted"]
    },
    "visibility": {
      "type": "object",
      "additionalProperties": false,
      "required": ["user_visible", "internal_only", "promotable"],
      "properties": {
        "user_visible": { "type": "boolean" },
        "internal_only": { "type": "boolean" },
        "promotable": { "type": "boolean" }
      }
    },
    "source_references": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["source_type", "source_id", "observed_at"],
        "properties": {
          "source_type": {
            "type": "string",
            "enum": ["event_log", "chat", "task", "note_pointer", "external_import"]
          },
          "source_id": { "type": "string", "minLength": 1 },
          "observed_at": { "type": "string", "format": "date-time" },
          "excerpt": { "type": "string" },
          "weight": { "type": "number", "minimum": 0, "maximum": 1 }
        }
      }
    },
    "corrections": {
      "type": "array",
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["status", "reason", "corrected_at"],
        "properties": {
          "status": {
            "type": "string",
            "enum": ["none", "superseded", "retracted", "amended"]
          },
          "reason": { "type": "string", "minLength": 1 },
          "corrected_at": { "type": "string", "format": "date-time" },
          "corrected_by": { "type": "string" },
          "replacement_entry_id": { "type": "string" }
        }
      }
    },
    "flags": {
      "type": "object",
      "additionalProperties": false,
      "required": ["user_visible", "internal", "promotable"],
      "properties": {
        "user_visible": { "type": "boolean" },
        "internal": { "type": "boolean" },
        "promotable": { "type": "boolean" }
      }
    },
    "version": { "type": "integer", "minimum": 1 },
    "metadata": {
      "type": "object",
      "additionalProperties": true
    }
  }
}
```

---

## Field Semantics

- `summary`: short narrative of what changed in Alice's understanding.
- `learned_about_user[]`: concrete user-model updates inferred from evidence.
- `help_strategy[]`: concrete next-help actions Alice should apply in this domain.
- `privacy_tier`: handling class for storage, retrieval, and downstream visibility.
- `source_references[]`: provenance pointers to raw evidence (do not require denormalized raw payload).
- `corrections[]`: annotation trail for post-hoc quality correction.
- `flags` + `visibility`: routing and promotion controls for UI and internal processors.

---

## Per-App Mapper Responsibilities

Each app owns a mapper that transforms app-native logs/interactions into the canonical `journal_entry`.

## 1) EVOtraining mapper

**Primary evidence**
- workout execution logs
- compliance/adherence events
- coach/user dialog snippets related to training behavior

**Must infer**
- consistency patterns, fatigue/risk cues, preference drift
- actionable training nudges (`help_strategy`)

**Special rules**
- any injury/pain signals default to `pt2_sensitive` minimum
- auto-generated corrections when later workout events invalidate earlier inference

## 2) EVOmind mapper

**Primary evidence**
- reflection conversations
- mood/state check-ins
- cognitive strategy usage markers

**Must infer**
- emotional regulation patterns
- strategy efficacy by context/time-of-day

**Special rules**
- mental-health-adjacent content defaults to `pt2_sensitive`
- mapper must avoid diagnostic language in `summary`

## 3) EVOlearn mapper

**Primary evidence**
- lesson/task completion data
- misunderstanding and remediation loops
- study cadence and retention signals

**Must infer**
- learning style preference, friction points, reinforcement needs

**Special rules**
- academic or credential-sensitive references must preserve provenance
- corrections should link to replacement entries when misconceptions are resolved

## 4) EVOconnect mapper

**Primary evidence**
- social/accountability interactions
- collaboration or check-in events
- community engagement patterns

**Must infer**
- support-network effectiveness
- social triggers that improve consistency

**Special rules**
- third-party identifiable content should elevate privacy tier appropriately
- include minimal excerpts in source references when peer privacy is implicated

---

## Correction Annotation Contract

Corrections are append-only annotations and SHOULD NOT mutate historical evidence.

- `none`: no correction state.
- `amended`: detail refined without invalidating core claim.
- `superseded`: replaced by newer understanding.
- `retracted`: original claim invalid/unsafe.

When `status in ["superseded", "retracted"]`, mappers SHOULD include `replacement_entry_id` when available.

---

## Source Reference Contract

Each mapper MUST provide traceable evidence references:

- stable `source_id`
- `source_type`
- `observed_at` timestamp
- optional `excerpt` and confidence `weight`

This enables downstream auditability and correction workflows without coupling journal storage to raw event schemas.

---

## Privacy Tiers

- `pt0_public`: safe for broad user-visible surfaces.
- `pt1_user`: user-private standard journal content.
- `pt2_sensitive`: heightened sensitivity (health/mental/emotional/safety signals).
- `pt3_restricted`: restricted processing path, explicit authorization required.

All consumers MUST enforce minimum-tier access checks prior to retrieval/rendering.

---

## Visibility and Promotion Flags

- `user_visible`: can surface directly in user-facing journal UI.
- `internal`: internal cognition/quality workflows only.
- `promotable`: candidate for promotion into higher-level synthesized memory artifacts.

`internal=true` SHOULD imply `user_visible=false` in policy enforcement.

---

## Implementation Readiness Checklist

- Canonical schema defined and validator-ready.
- Mapper obligations defined for all four apps.
- Corrections annotation model defined.
- Source references/provenance model defined.
- Privacy tier model defined.
- User-visible/internal/promotable flags defined.
- Journal/Living Notes separation made explicit.
- Aligned to EVO Cognition Layer doctrine.
