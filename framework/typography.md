# Typography

One family, a small scale, clear hierarchy. Define the scale in `ui/theme/Type.kt`
and reach for named roles — not arbitrary `sp` values scattered in screens.

## Family

- **System default sans** (Roboto / device sans) for all UI. No custom font files
  unless a brand reason demands it — system fonts render crisply at every density
  and cost nothing to ship.
- **Monospace** for anything column-aligned or "machine" — GPS coordinates, codes,
  numeric readouts. It signals precision and stops digits from dancing.

## The scale

| Role | Size | Weight | Colour | Use |
|------|------|--------|--------|-----|
| Display / brand | 28sp | Bold | gold | app title on home/hero |
| Title | 18–20sp | Bold/SemiBold | white | screen + section titles |
| Body | 14–16sp | Normal | white | primary content |
| Body strong | 14–16sp | Bold | white | the one line that matters (address) |
| Secondary | 12–14sp | Normal | white 70% / `TextSecondary` | captions, subtitle |
| Label / chip | 11–13sp | SemiBold/Bold | varies | chips, badges, headers |
| Mono | 12–14sp | Normal (mono) | `TextSecondary` | coordinates / codes |

## Rules that make text feel right

- **One bold thing per block.** Bold the single most important line (e.g. the
  address); keep the rest regular. Everything bold = nothing bold.
- **Cap wrapping.** Long dynamic text (addresses, notes) gets `maxLines` +
  `TextOverflow.Ellipsis` so a card can't grow unbounded.
- **Shadow over photos.** White text on a live camera/photo needs a subtle text
  `Shadow` (or a translucent backing) so it stays legible on any scene.
- **Tabular / monospace for numbers** that update in place — coordinates, timers,
  counters — so they don't reflow on each tick.
- **Letter-spacing only on small all-caps labels** (e.g. a header band) — ~0.08–0.15.
  Never on body.
- **Respect the user's locale.** Hindi (Devanagari) lines run taller than Latin —
  don't hard-code line heights that assume Latin metrics; let text measure itself.

## Burned-in vs on-screen text

Two different jobs, keep them consistent:
- **On-screen** (Compose) uses `sp` and the scale above.
- **Burned-into an image** (e.g. a photo stamp) scales off the image width
  (`size = base * imageWidth / REFERENCE_WIDTH`) so it stays readable at any
  resolution and matches across portrait/landscape. Keep the on-screen preview's
  *style* (header band, accent, fields) mirroring the burned result — WYSIWYG.
