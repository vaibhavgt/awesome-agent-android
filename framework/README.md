# tranquilwaters Android house style

The conventions every new tranquilwaters app starts from — so an agent (or a human)
building app number N already knows the design system, the type scale, the motion
rules, the tech stack, and the launch path **before writing a line of code**.

The main [README](../README.md) covers the *dev loop* (build → install → observe →
diagnose) that's true for any Android app. This folder covers the *house style* —
the opinionated defaults that make our apps look and feel like one family.

> Reference implementation: **Gps Camera Location** (`gpstools`) — **live on Google
> Play**. When a doc says "we do X", it means X shipped in a real, published app, not
> a wish. The "Field notes (from shipping)" sections capture what actually bit us.

## Read in this order

1. [tech-stack.md](tech-stack.md) — languages, libraries, versions, project shape
2. [design-system.md](design-system.md) — brand, colour roles, spacing, components
3. [typography.md](typography.md) — the type scale and when to use each step
4. [animation.md](animation.md) — motion principles + the small set of effects we reuse
5. [conventions.md](conventions.md) — package naming, signing, build types, locale
6. [monetization.md](monetization.md) — AdMob + Play Billing (IAP + subscription)
7. [launch-checklist.md](launch-checklist.md) — everything Play needs, in order
8. [promo-video.md](promo-video.md) — post-launch: a 16s store/ad video from the kit

## The house philosophy (one screen)

- **Calm, premium, utility-first.** Dark navy canvas, a single warm gold accent,
  generous spacing. Content over chrome.
- **One accent colour.** Navy is the surface; gold is the *only* highlight. If a
  second accent creeps in, something is wrong.
- **Edge-to-edge, system-aware.** Respect status/navigation bar insets everywhere.
- **Motion is subtle and purposeful.** A gentle pulse, a capture flash, a drawer
  slide. Never decorative spinning.
- **Free with ads, optional Pro.** Stacked monetization: banner + interstitial,
  a one-time remove-ads IAP, and a Pro subscription — all gated behind one flag.
- **India-first.** Hindi + English from day one; NavIC/▸regional where relevant.
- **Ship small, ship signed, ship to Play.** keystore.properties signing, a
  GitHub-Pages privacy policy, a repeatable store-listing kit.

## Brand in three values

| Token | Hex | Use |
|-------|-----|-----|
| Navy | `#15294D` | primary brand / surfaces |
| Navy deep | `#0F1626` | dark background / gradient bottom |
| Gold | `#F2A93B` | the one accent — titles, icons, selected state |

Everything else (greys, container navies, soft gold) derives from these — see
[design-system.md](design-system.md).
