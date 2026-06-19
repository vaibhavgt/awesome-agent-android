# Launch checklist

Everything Google Play needs, in the order it asks. This is the path we actually
walked to take an app from "builds locally" to "live on Play in ~1–2 days".

## 0. Before the first upload (permanent decisions)

- [ ] `applicationId = com.tranquilwaters.<app>` (see conventions — **permanent**)
- [ ] Release signing wired via `keystore.properties`; `release.keystore` backed up
- [ ] `versionCode` / `versionName` set (start `1` / `0.1.0` or similar)
- [ ] Real AdMob app id + unit ids in the `release` build type

## 1. Build the artifact

```
./gradlew clean bundleRelease assembleRelease
```
- Upload the **AAB** to Play (`app/build/outputs/bundle/release/app-release.aab`).
- Keep the signed APK for WhatsApp/sideload tests.

## 2. Store listing assets (the kit)

| Asset | Spec |
|-------|------|
| App icon | 512×512 PNG, 32-bit |
| Feature graphic | 1024×500 PNG/JPG |
| Phone screenshots | 2–8, framed + captioned, ~1080×1920 (real device > emulator) |
| Short description | ≤ 80 chars, keyword-led |
| Full description | ≤ 4000 chars, features + use-cases + keywords |
| App name | ≤ 30 chars |

House tip: generate the feature graphic + framed screenshots programmatically on the
navy+gold background (logo, title, caption per shot). Keep a `store/` folder in the
repo with all of them.

## 3. App content forms (all required)

- [ ] **Privacy policy** URL (GitHub Pages — see conventions)
- [ ] **Data safety** — declare honestly. For a camera/location app: *Precise
      location* + *Device or other IDs* (AdMob) collected & shared; photos stay
      on-device (not declared if never sent off-device); encrypted in transit; no
      account. (Don't over-declare; don't under-declare.)
- [ ] **Ads** — "Contains ads" = **Yes**
- [ ] **Content rating** questionnaire → usually Everyone
- [ ] **Target audience** — 18+ / not directed at children
- [ ] **App access** — note that all features work without login
- [ ] **Government / financial / health** declarations — usually N/A

## 4. Store settings

- [ ] App or game = **App**
- [ ] Category = **Tools** (or Photography) — pick where users actually search
- [ ] Tags + contact email

## 5. Release

- [ ] Production (or Internal testing first) → Create release → upload AAB
- [ ] Accept **Play App Signing** on first upload
- [ ] Release notes (short, bullet the headline features)
- [ ] Countries — all (176+) unless there's a reason not to
- [ ] **Send for review** → quick checks run, then Google reviews

## 6. After review

- First-app review: a few hours to ~7 days (often 1–3). Approve → **live**.
- Ads warm up over ~1–2 days post-publish (no fill before that is normal).
- Share the listing link for first installs/reviews; watch early ratings.
- If rejected: read the policy reason, fix, resubmit — don't guess.

## Reference: the assets that shipped

A real launch (`gpstools`) shipped: `ic_play_store_512.png`,
`store/feature-graphic-1024x500.png`, three framed `store/screenshot-*.png`,
`docs/index.html` privacy policy on Pages, a signed AAB, and a `PLAY-RELEASE.md`
in-repo capturing fingerprints + this checklist. Copy that structure per app.

## Field notes (from shipping)

- **Updating the icon? Update the listing graphics too.** The launcher icon (in the
  APK) and the Play **512 store icon** + **feature graphic** are separate. If you
  redraw the logo, regenerate and re-upload the 512 icon and any graphic that contains
  it — otherwise the store shows the old mark.
- **Build and store-listing changes are separate submissions.** In *Publishing
  overview* a new app build (Production release) and graphic/text edits show as
  distinct pending items. Bundle them into **one** "Submit for review" so the live app
  and listing update together — a graphics-only submit leaves the old build live (and
  e.g. new IAP product ids won't be in the live code yet).
- **Discarded a release draft? Bump versionCode before rebuilding** to avoid a
  "version code already used" error on re-upload.
- **applicationId is permanent** — see conventions. Decide `com.<org>.<app>` before
  the first upload; you cannot change it later without a brand-new listing.
