# Tech stack

The default stack for a new tranquilwaters app. Don't re-litigate these per project
unless there's a real reason — sameness is the point.

## Language & build

- **Kotlin**, 100% — no Java sources.
- **Jetpack Compose** for all UI. No XML layouts (XML only for adaptive-icon /
  splash resources).
- **Gradle Kotlin DSL** (`build.gradle.kts`) with a **version catalog**
  (`gradle/libs.versions.toml`). Reference deps as `libs.xxx`, never hardcoded
  coordinates in the module file.
- Always use the project's `./gradlew` wrapper.

## Versions (known-good baseline)

These are the versions the reference app ships. Bump deliberately, together.

| Thing | Version |
|-------|---------|
| Kotlin | 2.0.21 |
| AGP | 8.5.2 |
| Compose BOM | 2024.10.01 |
| Gradle | 8.9 |
| Java / JVM target | 17 |
| minSdk | 24 |
| targetSdk / compileSdk | 35 |
| build-tools | 35.0.0 |

`minSdk 24` is the floor — it covers ~99% of active devices and unlocks adaptive
icons, vector drawables and modern APIs without legacy workarounds.

## Default libraries

Pull these in only when the app needs them, but when it does, use *these*:

| Need | Library |
|------|---------|
| UI | Compose Material3 (`androidx.compose.material3`) |
| Icons | `material-icons-extended` |
| Navigation | `androidx.navigation:navigation-compose` |
| Runtime permissions | `accompanist-permissions` |
| Camera | CameraX (`camera-core/camera2/lifecycle/view`) |
| Location | `play-services-location` (FusedLocation) |
| Maps (free) | `osmdroid` (OSM tiles, no API key) |
| Image loading | `coil-compose` |
| Ads | `play-services-ads` (AdMob) |
| Billing | `billing-ktx` (Play Billing) |
| EXIF | `androidx.exifinterface` |

Notes that bite:
- `play-services-ads` drags in full Guava, which makes Gradle dedupe CameraX's
  `listenablefuture` to the empty stub. If you use **both** ads and CameraX,
  declare `guava` directly so `ProcessCameraProvider` resolves.
- Free-tier maps + weather: **osmdroid** (tiles) + **Open-Meteo** (weather) — both
  keyless. Reach for a keyed provider (Google/Mapbox) only if you need satellite.

## Project shape

```
app/src/main/java/com/tranquilwaters/<app>/
  MainActivity.kt            // edge-to-edge, theme, nav host, billing init
  ui/
    theme/    Color.kt Theme.kt Type.kt
    navigation/ Destinations.kt
    screens/  one file per screen + shared overlays
  <domain>/   camera/ location/ media/ settings/ …
  ads/        Ads.kt BannerAd.kt InterstitialAdManager.kt
  billing/    BillingManager.kt Premium.kt Subscription.kt
  locale/     in-app language wrapper
```

- One screen = one Composable file. Shared pieces (cards, overlays) live beside them.
- `MainActivity` does four things: `enableEdgeToEdge()`, apply theme, host nav,
  init ads + billing (+ restore entitlements).

## Architecture stance

- **State hoisting + plain Compose state.** No DI framework, no ViewModel ceremony
  for small apps — `remember`/`rememberSaveable` and small stores. Add Hilt/Room
  only when an app is genuinely data-heavy.
- **Tiny SharedPreferences-backed stores** for settings/entitlements (one class each).
- **Everything off the main thread** that touches disk/network/bitmaps — capture,
  geocode, tile fetch, PDF export all run on IO and post results back.
