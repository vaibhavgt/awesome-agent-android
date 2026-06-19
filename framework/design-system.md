# Design system

Calm, premium, utility-first. A dark navy canvas with **one** warm gold accent.

## Colour

Define these once in `ui/theme/Color.kt`. Never use raw hex at call sites — always
a named token.

| Token | Hex | Role |
|-------|-----|------|
| `BrandNavy` | `#15294D` | primary brand, top bars, selected chip fill |
| `BrandNavyDeep` | `#0F1626` | screen background / gradient bottom |
| `BrandNavySurface` | `#131C2E` | cards / surfaces on the dark canvas |
| `BrandNavyLight` | `#2A4470` | containers, icon circles, dividers |
| `BrandGold` | `#F2A93B` | **the accent** — titles, icons, selected, CTAs |
| `BrandGoldDark` | `#C98A24` | gold on light backgrounds (contrast) |
| `BrandGoldSoft` | `#FFD699` | soft gold for dark-theme tertiary |
| `TextSecondary` | `#9AA0A6` | muted labels, coordinates, captions |

Rules:
- **Gold is the only accent.** It marks what's interactive or important. If you reach
  for a second highlight colour, reconsider the hierarchy instead.
- White at **70% alpha** for secondary text on navy; full white for primary.
- Status colours (success green, warn amber, error red) are *functional*, used only
  for state (e.g. GPS accuracy chip), never as decoration.
- Home/landing uses a 3-stop vertical gradient: a slightly lighter navy
  (`#1B3A66`) → `BrandNavy` → `BrandNavyDeep` for depth.

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
