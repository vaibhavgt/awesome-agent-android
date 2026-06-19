# Animation & motion

Motion exists to **explain, confirm, and delight — in that order**. If a motion
doesn't tell the user something (where they are, what just happened, what's coming),
cut it. We're a utility brand; calm beats flashy.

## Principles (the 12, distilled for an app)

- **Ease, never linear.** Real things accelerate and settle. Use
  `FastOutSlowInEasing` (standard), or a spring for anything draggable/physical.
  Linear motion reads as cheap.
- **Anticipation & follow-through** for big moves — a slight overshoot/settle on an
  entrance feels alive; a hard stop feels mechanical. Keep it subtle.
- **Staging** — one thing draws the eye at a time. Don't animate five elements at once.
- **Secondary action** — small supporting motion (a thumbnail updating after capture)
  reinforces the primary action without stealing focus.
- **Slow in / slow out + arcs** — natural paths, not robotic straight lines.
- **Appeal** — purposeful, branded, restrained. A gold pulse, not a rainbow.

## Durations & curves

| Motion | Duration | Curve |
|--------|----------|-------|
| Micro (tap feedback, ripple) | 100–150ms | standard |
| Standard (fade/slide, sheet, drawer) | 200–300ms | FastOutSlowIn |
| Entrance (content settling in) | 300–400ms | FastOutSlowIn / gentle spring |
| Ambient loop (breathing logo) | 1500–2000ms each way | FastOutSlowIn, Reverse |

Rule of thumb: **if you notice the duration, it's too long.** Functional motion is
felt, not watched.

## The reusable effect set (what we actually ship)

These are house effects — reuse them, don't reinvent:

1. **Breathing hero** — the home logo scales `1.0 → 1.06` on an infinite
   `Reverse` repeat (~1600ms). Gives a static landing some life; fills negative space.
2. **Capture flash** — a brief full-screen white flash on shutter press. The
   universal "photo taken" cue; pair with a progress ring so a tap is never ambiguous.
3. **In-place progress** — a small ring *inside* the shutter while saving, so the
   user knows work is happening without a blocking dialog.
4. **Drawer slide** — `ModalNavigationDrawer`'s built-in slide; don't customize it.
5. **Thumbnail refresh** — the gallery thumbnail updates to the just-captured photo
   (secondary action confirming success).
6. **Live preview reactivity** — when a choice changes (e.g. stamp style), the
   on-screen preview updates *immediately* via Compose state. If the output changes
   but the preview doesn't, that's a bug, not a missing animation.

## Don'ts

- **No gratuitous spinning** of logos/icons (a rotating location pin looks broken).
- **No motion that blocks input** — never gate a capture/CTA behind an animation or
  an ad. Action first, motion/ads after.
- **No animating on every recomposition** — drive motion from explicit state, not
  side-effects that fire repeatedly.
- **Respect reduced-motion** where the platform exposes it; keep ambient loops gentle
  enough that they're never a problem.

## Tools

- `animateFloatAsState` / `animate*AsState` for simple value transitions.
- `rememberInfiniteTransition` for ambient loops (the breathing hero).
- `Animatable` for one-shot, controllable sequences (the capture flash).
- `AnimatedVisibility` / `Crossfade` for enter/exit and swaps.
- Audit motion against these with the `12-principles-of-animation` and
  `make-interfaces-feel-better` skills when polishing.
