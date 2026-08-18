---
title: Alice Visibility Rules
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-mirrors/Alice Visibility Rules.md"]
updated: 2026-07-24
---

# Alice Visibility Rules
[Alice Visibility Rules](https://www.notion.so/33ec72bad01381cda0d9f5eb01c608bb)
Concept
Alice’s visual and conversational presence is context-dependent.
Rule / Mechanism
Alice appears only during: - Meaningful feedback moments - Summaries - Decisions requiring explanation - Explicit user interaction
She remains invisible during routine execution.
Why It Exists
Users should feel supported, not monitored.
Implications
Cleaner UI
Reduced cognitive load
Stronger moments of impact
See also: [[Presence by Value]], [[EVOsystem — Alice Presence Model]]

---

## Canonical Presentation System (Flutter)

Alice's canonical visual system is the **layered PNG avatar** (`AlicePngAvatar`) in `flutter_app/assets/alice/`. This is the only fully implemented cross-surface presentation stack in active Flutter use.

### Asset Set (15 canonical files)

All in `flutter_app/assets/alice/`:

| Layer | File | Notes |
|---|---|---|
| Head | `head.png` | Base silhouette |
| Torso | `torso.png` | Body core |
| Arms | `arm.png` | Reused mirrored (L/R) |
| Chest light | `chest_light.png` | Beam origin for projections |
| Headphones | `headphones.png` | StrainSync contexts only |
| Eyes (neutral) | `open.png` | Default eye state |
| Eyes (blink) | `blink.png` | Internal blink timer |
| Eyes (wink) | `wink_left.png` | On tap |
| Eyes (angry) | `angry.png` | Emotion variant |
| Eyes (sad) | `sad.png` | Emotion variant |
| Mouth (neutral) | `smile.png` | Default |
| Mouth (excited) | `big_smile.png` | Emotion variant |
| Mouth (sad) | `frown.png` | Also: disappointed |
| Mouth (angry) | `subtle_angry.png` | Emotion variant |
| Mouth (surprised) | `o_surprised.png` | Emotion variant |

### Projection System

- `AliceHologramProjection` and `AliceHologramBeamCardOverlay` are reusable primitives
- Beam origin/target computation still lives inside Training screens — extraction target for future
- Recommended extraction boundary: `packages/ui/src/alice/` with explicit asset manifest

### Non-canonical renderers (do not use as extraction baseline)

`AliceBlobAvatar`, `AliceSvgBlobAvatar`, `AliceAnimatedSvgAvatar`, `AliceStatusAvatar`, `AliceHeadWidget` — present in repo but do not reconstruct the layered Training Alice.

### iOS Widget duplicate assets (deprecated)

Widget extension copies in `flutter_app/ios/EvoFitnessWidget/Assets.xcassets/` are byte-identical to the canonical Flutter set — marked deprecated / duplicate. The shared extraction should eliminate these.

## Related

^[{src_rel}]
