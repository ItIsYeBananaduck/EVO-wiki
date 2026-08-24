---
title: ALICE_AVATAR_USAGE_AUDIT
type: concept
tags: ["EVO","doctrine"]
sources: ["EVO/smartdocs/raw/deprecated/doctrine-unmatched/ALICE_AVATAR_USAGE_AUDIT.md"]
updated: 2026-07-24
---

# Alice Avatar Usage Audit

## Scope and method

This audit covers the Alice avatar assets that are actually present in the repository and appear to be intended for UI rendering. I classified an asset as:

- **In use** when it is referenced by a widget/view that is instantiated from another runtime screen or target.
- **Not currently in use** when I could only find tooling, docs, packaging, or test references, or when the owning widget has no call sites.
- **Partially used / support-only** when the asset is real and bundled, but the only confirmed use is preview/test/tooling rather than a production UI path.

## Summary

### In use

| Asset or avatar                                                                                                                                                                                                                                                     | Where it is used                                                                                           | Assessment |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ---------- |
| `flutter_app/assets/alice-animated.svg`                                                                                                                                                                                                                             | Loaded by `AliceAnimatedSvgAvatar`, which is rendered in `trainer_desktop_view.dart`                       | **In use** |
| `flutter_app/assets/alice/` PNG parts (`head.png`, `torso.png`, `arm.png`, `open.png`, `blink.png`, `smile.png`, `frown.png`, `sad.png`, `big_smile.png`, `o_surprised.png`, `subtle_angry.png`, `wink_left.png`, `chest_light.png`, `headphones.png`, `angry.png`) | Used by `AlicePngAvatar` and `AliceHeadWidget`, both of which are instantiated from active Flutter screens | **In use** |
| `flutter_app/ios/EvoFitnessWidget/Assets.xcassets/Alice*.imageset/*.png` (`AliceHead`, `AliceTorso`, `AliceChestLight`, `AliceArm`, `AliceOpen`, `AliceBlink`, `AliceSmile`, `AliceFlat`)                                                                           | Used by `WorkoutLiveActivity.swift` in the iOS widget/live activity target                                 | **In use** |

### Not currently in use

| Asset or avatar                                                      | Evidence                                                                                                                                  | Assessment               |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| `flutter_app/assets/alice-blob-no-legs.svg`                          | Only referenced by `AliceSvgBlobAvatar`, and that widget has no call sites                                                                | **Not currently in use** |
| `flutter_app/lib/features/alice/presentation/alice_blob_avatar.dart` | The widget exists, but no runtime usage was found                                                                                         | **Not currently in use** |
| `app/static/alice.svg`                                               | No runtime app references found; only the Flutter copy is bundled and the Flutter test reads `assets/alice.svg` from `flutter_app/assets` | **Not currently in use** |
| `app/static/alice-animated.svg`                                      | Referenced by optimization scripts, but I found no runtime web view that loads it                                                         | **Not currently in use** |
| `app/static/assets/Character_output-opt.glb`                         | Referenced by an optimization script as input; the watch 3D loading path is commented out and falls back to 2D                            | **Not currently in use** |
| `app/static/alice-preview.html`                                      | Standalone preview file with no inbound references found                                                                                  | **Not currently in use** |

### Support-only / limited use

| Asset or avatar                    | Evidence                                                                                                                         | Assessment       |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| `app/static/alice-transparent.png` | Used by `app/static/alice-preview.html`, but that preview page itself does not appear to be linked from production code          | **Support-only** |
| `flutter_app/assets/alice.svg`     | Bundled in `pubspec.yaml` and used by `alice_asset_download_manager_test.dart`, but I found no production UI loading it directly | **Support-only** |

## Detailed findings

### 1. Active Flutter avatar: animated SVG

`AliceAnimatedSvgAvatar` loads `assets/alice-animated.svg` from the Flutter bundle. That widget is instantiated in `trainer_desktop_view.dart`, so the Flutter copy of `alice-animated.svg` is definitely live.

### 2. Active Flutter avatar: layered PNG Alice

`AlicePngAvatar` builds Alice from the `assets/alice/` PNG part set. It is rendered on the home screen, the live workout screen, and the trainer live workout view. Separately, `AliceHeadWidget` uses the same PNG part family for Alice chat bubbles in `conversation_mode_screen.dart`. That means the PNG part library under `flutter_app/assets/alice/` is actively used.

### 3. Active iOS widget/live-activity avatar

The iOS live activity/widget target checks for and renders `AliceFlat`, `AliceHead`, `AliceTorso`, `AliceChestLight`, `AliceArm`, `AliceOpen`, `AliceBlink`, and `AliceSmile`. Those image-set assets are therefore active for the iOS widget/live activity surface.

### 4. Unused Flutter avatar variants

`AliceSvgBlobAvatar` loads `assets/alice-blob-no-legs.svg`, but I found no call sites for `AliceSvgBlobAvatar`. Likewise, `AliceBlobAvatar` exists as a separate painted implementation, but it also has no call sites. Both appear to be inactive alternatives at the moment.

### 5. Web/static Alice assets that appear inactive

The web/static directory contains several Alice assets, but I did not find runtime code loading them:

- `app/static/alice.svg` has no confirmed production use.
- `app/static/alice-animated.svg` is referenced by SVG optimization scripts only.
- `app/static/assets/Character_output-opt.glb` is referenced by the watch optimization script, while the current watch `Alice3DView.swift` explicitly falls back to 2D and leaves the 3D loading block commented out.
- `app/static/alice-preview.html` appears to be a manual preview page rather than part of a navigated app flow.

### 6. Preview/test-only assets

`app/static/alice-transparent.png` is still used, but only by `app/static/alice-preview.html`, which makes it a preview/support asset rather than a confirmed production avatar. Similarly, `flutter_app/assets/alice.svg` is bundled and used by the asset download manager test, but I found no production widget that displays it directly.

## Recommended follow-up

1. **Keep** the active Flutter assets:
   - `flutter_app/assets/alice-animated.svg`
   - `flutter_app/assets/alice/`
   - iOS widget `Alice*.imageset` assets
2. **Review for cleanup or archival**:
   - `flutter_app/assets/alice-blob-no-legs.svg`
   - `flutter_app/lib/features/alice/presentation/alice_blob_avatar.dart`
   - `app/static/alice.svg`
   - `app/static/alice-animated.svg`
   - `app/static/assets/Character_output-opt.glb`
   - `app/static/alice-preview.html`
3. **Keep only if the preview/test workflows still matter**:
   - `app/static/alice-transparent.png`
   - `flutter_app/assets/alice.svg`

## Notes / caveats

- This is a **repository reference audit**, not a runtime instrumentation audit.
- I treated commented-out code, packaging logs, and docs as **insufficient** evidence of active use.
- I excluded app icons such as `alicenew.png` from the main used/unused judgment because they are branding/app-icon assets, not in-app Alice avatar renderers.

## Related
