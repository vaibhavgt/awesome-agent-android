# Design system

Calm, premium, utility-first. A dark navy canvas with **one** warm gold accent.

> **Method, not just a palette.** For the *design-thinking* process — interrogating
> purpose/users/tone before committing, picking a bold direction, and avoiding generic
> "AI-slop" aesthetics — use the vendored
> [`bencium-innovative-ux-designer`](../skills/bencium-innovative-ux-designer/SKILL.md)
> skill. This doc is the **house defaults** (navy + gold, our components); the skill is
> the **reasoning** you apply on top.

## Colour

**Single source of truth: `ui/theme/Color.kt`.** Hex values live there and *nowhere
else* — this doc references tokens by **role**, never literals, so the palette can
change in one place without the docs going stale. At call sites, always use a named
token, never raw hex.

| Token | Role |
|-------|------|
| `BrandNavy` | primary brand, top bars, selected chip fill |
| `BrandNavyDeep` | screen background / gradient bottom |
| `BrandNavySurface` | cards / surfaces on the dark canvas |
| `BrandNavyLight` | containers, icon circles, dividers |
| `BrandNavyGradientTop` | the lighter navy at the top of the home/landing gradient |
| `BrandGold` | **the accent** — titles, icons, selected, CTAs |
| `BrandGoldDark` | gold on light backgrounds (contrast) |
| `BrandGoldSoft` | soft gold for dark-theme tertiary |
| `TextSecondary` | muted labels, coordinates, captions |

> The three brand values (navy / navy-deep / gold) are quoted once in the framework
> [README](../README.md#brand-in-three-values) as the canonical reference; everything
> else derives from them in `Color.kt`.

Rules:
- **Gold is the only accent.** It marks what's interactive or important. If you reach
  for a second highlight colour, reconsider the hierarchy instead.
- White at **70% alpha** for secondary text on navy; full white for primary.
- Status colours (success green, warn amber, error red) are *functional*, used only
  for state (e.g. GPS accuracy chip), never as decoration.
- Home/landing uses a 3-stop vertical gradient:
  `BrandNavyGradientTop` → `BrandNavy` → `BrandNavyDeep` for depth.

## Spacing & shape

- Base unit **4dp**. Common steps: 4 / 8 / 12 / 16 / 24 / 32.
- Screen edge padding **16dp** (24dp for hero/landing).
- Card corner radius **16dp** (20dp for large tiles, 10–12dp for thumbnails/chips).
- Card elevation **4dp**; modal/sheet uses Material default.
- Group related items with `Arrangement.spacedBy(...)`, not manual spacers, inside rows/cols.

## Layout & insets

- **Edge-to-edge always** (`enableEdgeToEdge()` in the activity).
- Respect insets at the screen edges: `statusBarsPadding()` at the top,
  `navigationBarsPadding()` at the bottom. A bug we actually shipped once: a fixed
  `height()` on a bottom bar swallowed the gesture-bar inset and clipped labels —
  let components size themselves and add inset padding instead.
- Content sits in the **upper-middle**, not pinned to the very top (empty bottom) or
  dead-centre (empty top). Flexible weighted spacers above/below, content slightly
  above centre.

## Core components (the kit)

| Component | House treatment |
|-----------|-----------------|
| **App bar** | navy `TopAppBar`; gold back/menu icon; white title. Camera screens use a translucent overlay bar instead of a solid one. |
| **Navigation** | `ModalNavigationDrawer` (☰ top-left), navy sheet, gold icons, white labels. No bottom nav. |
| **Card / tile** | `BrandNavySurface`, 16–20dp radius, gold icon in a `BrandNavyLight` circle, white label. |
| **Chip / selector** | prefer a **visual preview** over a text chip when the choice is visual (e.g. stamp styles render a real thumbnail). |
| **Bottom sheet** | `ModalBottomSheet` for editing/options; group "pick a style" + "details" together rather than scattering controls. |
| **Banner ad** | pinned to the bottom edge, above the nav-bar inset; self-hides for Pro. |

## Iconography & launcher icon

- Material **filled** icons, gold-tinted on navy.
- Launcher icon: **flat, vector-style, drawn (not AI-rendered photos)** — a clean
  navy mark + gold accent, no baked drop-shadows or grey rims. Generate **all**
  adaptive densities + a 512 store icon from one source.
- Branded **splash**: set `android:windowBackground` to a white drawable with the
  centred logo (covers *all* API levels), and `windowSplashScreenBackground` (white)
  in `values-v31` for the Android-12+ system splash. Don't rely on the platform
  default — it tints the pre-draw window an off-brand colour.

## Anti-patterns (we hit these, don't repeat)

- A second accent colour competing with gold.
- Always-visible control rows that crowd the screen — move secondary controls into
  the adjust/▸options sheet.
- Pinning content to the top → big empty bottom. Centre it.
- Forgetting insets → clipped content behind the system bars.
