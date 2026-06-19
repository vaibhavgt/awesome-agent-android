# Project conventions

The non-design defaults: naming, signing, build types, locale, and the autonomous
build loop. Get these right at `app:new` time — several are **permanent** once you
ship.

## Package / applicationId — pick once, forever

- **Always `com.tranquilwaters.<appname>`.** This is the org namespace; keep every
  app under it for a coherent portfolio.
- ⚠️ **The applicationId is permanent on Google Play.** You cannot change it after
  publishing — a different id means a brand-new listing (new reviews, new install
  base). Decide it *before* the first upload.
  - Lesson learned the hard way: one app shipped as `com.gpstools.camera` instead of
    `com.tranquilwaters.gpscamera`. Users never see the id, so it was left as-is —
    but it broke the portfolio pattern. Don't repeat: set
    `com.tranquilwaters.<app>` from the first commit.

## Release signing

- Keep the keystore **out of git**. Signing config reads a gitignored
  `keystore.properties`:
  ```
  storeFile=release.keystore
  storePassword=…
  keyAlias=…
  keyPassword=…
  ```
- `build.gradle.kts` loads it and applies a `release` signingConfig **only when the
  file exists** (so CI/clones without the key still build, just unsigned).
- Result: `./gradlew assembleRelease` / `bundleRelease` emit a **signed** APK/AAB
  directly — no manual `zipalign`/`apksigner`.
- Enrol in **Play App Signing** on first upload; `release.keystore` becomes the
  *upload* key. **Back up `release.keystore` + `keystore.properties`** somewhere
  safe — losing them means you can never update the app.

## Build types carry the secrets

Per-build-type config keeps debug safe and release real:

- **AdMob ids** split by build type: `debug` uses Google's **TEST** ids, `release`
  uses the **real** ids. App id via a `manifestPlaceholders["admobAppId"]`; unit ids
  via `buildConfigField(...)`. The manifest reads `${admobAppId}`; code reads
  `BuildConfig.ADMOB_*`. This means an emulator/dev build can never serve (or
  accidentally click) a real ad.
- Enable `buildConfig = true` to use BuildConfig fields.
- DEBUG-only affordances (e.g. "simulate purchase") gate on `BuildConfig.DEBUG`.

## Localisation (India-first)

- Ship **English + Hindi** from the start: `values/strings.xml` + `values-hi/strings.xml`.
  Every user-facing string is a resource — no hardcoded text.
- In-app language switch: wrap the activity's base context with the stored locale
  (`attachBaseContext` → `wrapWithStoredLocale`) so the choice applies app-wide and
  can follow the device.
- Make the launcher activity `AppCompatActivity`-compatible for locale switching to
  actually take effect (a no-op toggle is a classic bug).

## Edge-to-edge & system

- `enableEdgeToEdge()` in `onCreate`. Then handle insets per screen
  (`statusBarsPadding`, `navigationBarsPadding`). See design-system for the gotcha.

## Privacy policy (required for camera/location apps)

- Host a simple static policy on **GitHub Pages**: put `docs/index.html` in the repo,
  enable Pages (Settings → Pages → main `/docs`). URL:
  `https://<user>.github.io/<repo>/`. Paste it into the Play listing.
- The policy must name what you access (precise location, camera, photos), where it
  goes (on-device; OSM/weather/AdMob third parties), and a contact email.

## Autonomous build loop (Ralph)

- New apps are scaffolded and built story-by-story by the **Ralph** loop
  (`scripts/ralph/`): a `prd.json` of `passes:false` stories, a per-iteration fresh
  agent that implements the highest-priority story, runs `./gradlew assembleDebug` as
  the quality gate, commits `feat: [ID] - [title]`, flips `passes:true`, and emits
  `<promise>COMPLETE</promise>` when done.
- Because it commits per story, any kill/hang/crash loses nothing — it resumes from
  `prd.json` state. Pin `ANDROID_SERIAL` when both a phone and emulator are attached,
  and run the emulator with hardware GPU (`-gpu host`) to avoid software-render ANRs.
- Feed Ralph this `framework/` folder so it starts every app already knowing the
  house style.
