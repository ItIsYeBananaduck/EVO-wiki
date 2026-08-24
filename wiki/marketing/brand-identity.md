---
title: Brand Identity
type: concept
description: EVOtraining visual identity — color scheme, logo, tagline, and Alice's canonical appearance.
tags: [evo, marketing, brand, identity, alice, color]
sources: []
updated: 2026-08-24
---

# Brand Identity

Canonical visual identity for EVOtraining. Source of truth for any asset, page, or content that represents the brand.

---

## Tagline

**TRAINING EVOLVED**

All uppercase in logo lockups. Sentence case acceptable in body copy.

### Launch language

Use **"Training evolves in 2027."** — not a specific launch date. Date-free framing means the message survives schedule slip or an early ship.

Avoid: "Launching New Year's Day," "Launching January 1," or any fixed date in evergreen assets. Fixed dates are fine in time-boxed campaign content that will be retired.

---

## Color Scheme

### Brand core

| Role | Hex | Notes |
|---|---|---|
| Primary purple | `#8a2be2` | Wordmark, primary CTAs, accents |
| Purple hover | `#9d4edd` | Interactive hover state |
| Cyan | `#00e5ff` | Alice's eyes, glow, secondary accent, tagline text |
| Magenta | `#ff0099` | Gradient terminus only — never a solid fill |

### Surfaces

| Role | Hex | Notes |
|---|---|---|
| Background | `#120524` | Deep purple-black, page base |
| Surface | `#1a0b2e` | Cards, panels |
| Surface raised | `#22103a` | Elevated elements |
| Border | `#35205c` | Dividers, card edges |
| Text | `#e8e8ec` | Body copy |
| Text muted | `#8a8a96` | Secondary copy |
| Success | `#4ade80` | Confirmation states only |

### Signature gradient

`linear-gradient(90deg, #00e5ff 0%, #8a2be2 55%, #ff0099 100%)`

Cyan → purple → magenta, left to right. Taken from the glowing ring in the logo. Use for accent headline text and emphasis moments. Do not use as a large background fill — it loses its impact when overused.

### Rules

- Dark backgrounds are the default. The palette is built for dark surfaces; on light backgrounds the neon values vibrate and lose legibility.
- Never use orange, teal, or generic tech blue. The original landing page draft used orange `#ff6b35` — that is retired and wrong.
- Magenta is a gradient endpoint, not a standalone brand color.

---

## Logo

The mark is a circular badge: Alice's head peeking over a large letter **E** flanked by dumbbell icons, wordmark and tagline below, all inside a cyan→magenta glowing ring on a deep purple field.

### Assets

| File | Size | Use |
|---|---|---|
| `assets/evotraining-logo.png` | 512×512 | Primary — transparent circular badge |
| `assets/logo-512.png` | 512×512 | Open Graph / social share |
| `assets/logo-180.png` | 180×180 | Apple touch icon |
| `assets/logo-32.png` | 32×32 | Favicon |
| `assets/evotraining-logo.jpg` | 458×465 | Original source (square background baked in) |

The PNGs were cut from the source JPG with a feathered circular alpha mask. A luminance or max-channel key does not work on this artwork — the background purple sits too close in value to the logo purple for any clean threshold. Circular masking is the correct approach.

### Rules

- Use the transparent PNG, not the JPG, anywhere the background is not the exact source purple.
- Keep the badge circular. Do not crop to a square, and do not place it on a contrasting light panel.
- Do not recolor the ring, the robot, or the wordmark.

---

## Alice — Canonical Appearance

Alice is the AI trainer and the face of the brand. Her appearance must stay consistent across every asset, illustration, and generated image.

### Body plan

| Element | Specification |
|---|---|
| **Head** | Large rounded square (squircle), slightly wider than tall. Helmet-like. No antenna — the crown is smooth. |
| **Face screen** | Black rounded rectangle inset in the head, roughly 60% of the face area, rounded corners. Near-black with blue undertone, not pure black. |
| **Eyes** | Two vertical pill/capsule shapes in bright cyan. Positioned slightly above the faceplate's horizontal center. |
| **Mouth** | Single upward-curving cyan line. Simple smile, centered below the eyes. |
| **Neck** | Short, distinct separation between head and torso. |
| **Torso** | Rounded bulbous bean or teardrop shape — narrower at the top, wider at the bottom. Lavender. |
| **Chest emblem** | One solid bright cyan circle, upper-center of the torso. Reads as a power core. |
| **Arms** | Two short curved appendages from the upper sides of the torso, curving inward and down — hook or "C" shaped. No hands, no forearms. Dark purple, matching the head casing. |
| **Legs** | None. She floats. |
| **Pose** | Frontal with depth. Arms curved inward in a welcoming, near-hugging position. |
| **Lighting** | Soft glow from below, purple haze around her. Reads as hovering. |

### Alice's colors

These differ from the brand UI palette — she is rendered art, not interface.

| Element | Hex |
|---|---|
| Head casing | `#4a3b6b` (deep muted indigo-purple) |
| Torso | `#8b7cb5` (medium lavender) |
| Eyes, mouth, chest core | `#00eaff` (neon cyan) |
| Face screen | `#0f0f18` (blue-black) |
| Background inner glow | `#2d1f4e` (hazy deep violet) |
| Background outer | `#020105` (near-black, purple tint) |

### Rules

- **No antenna.** The logo mark shows a small nub on her left side; the full-body render has none. Full-body Alice is smooth-crowned.
- **No legs, no hands.** She floats and her arms are simple curved stubs.
- Cyan is reserved for her glowing features — eyes, mouth, chest core. Do not use cyan elsewhere on her body.
- Keep her friendly. The design intent is approachable and non-intimidating, which is the whole point: she is the trainer who does not judge you.

---

## Voice Alignment

The visual identity carries the same message as the copy:

| Visual choice | What it signals |
|---|---|
| Dark purple, not gym-poster black-and-neon-green | Calm, private, not aggressive |
| Alice is round and smiling, not a chiseled avatar | No judgment, no intimidation |
| Cyan glow, soft lighting | Intelligent, alive, warm |
| Floating, no legs | Approachable AI, not a humanoid replacement for a person |

Alice looks the way the product is supposed to feel. A beginner too embarrassed to hire a trainer should look at her and feel relief, not pressure.

---

## Related

- [[EVO Marketing]]
- [[Messaging]]
- [[Positioning]]

^[EVO/wiki/marketing/brand-identity.md]