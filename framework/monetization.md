# Monetization

**Stacked, non-intrusive.** Free with ads → optional one-time "remove ads" →
optional Pro subscription. Everything routes through a single entitlement flag so
turning off ads is one call.

## The model

| Tier | What | Product type |
|------|------|--------------|
| Free | Banner + occasional interstitial | AdMob |
| Remove ads | One-time purchase, kills all ads | Play Billing **in-app product** |
| Pro | Power features (e.g. unlimited/▸watermark-free exports, batch) | Play Billing **subscription** |

One flag (`Premium`/`Subscription`) gates everything. Banners/interstitials check it
and render nothing when the user has paid.

## AdMob

- **Banner**: pinned to the bottom edge (above the nav-bar inset). Renders nothing
  when ads are disabled, so paid users see a clean bottom.
- **Interstitial**: shown *after* the Nth completed action (e.g. every 5th capture),
  **never before/▸blocking it**. Preload up front; show post-action.
- **Ids are per build type** (see conventions): TEST in debug, real in release. App
  id via manifest placeholder, unit ids via BuildConfig.
- **Policy, memorize:** real ads only serve once the app is **published + the AdMob
  account is approved**. An unpublished release shows an *empty* banner (no fill) —
  that's normal, not a bug; it fills 1–2 days after going live. **Never click your
  own live ads** (account ban). Register your device as a test device while testing.

## Play Billing — one-time IAP (remove ads)

- Connect on launch and **restore** entitlement (`queryPurchases`) so reinstalls
  re-grant automatically, no user action.
- Flow: `queryProductDetails` (INAPP) → `launchBillingFlow` → `onPurchasesUpdated`
  → **acknowledge** → set the `Premium` flag (persisted) → ads vanish via the flag.
- Acknowledge within 3 days or Play auto-refunds. Always acknowledge.

## Play Billing — subscription (Pro)

This is the bit most likely to be a stub at first. To make it real:

**1. Create the product in Play Console** (Monetize → Subscriptions):
- Prefer **one subscription** (e.g. `tw_pro`) with **multiple base plans**
  (monthly + yearly) — the modern Billing v6/v7 model. (Legacy alternative: two
  separate subscription products `pro_monthly` / `pro_yearly`.) Match whatever the
  code queries.
- Set prices per market (₹ for India; Play auto-converts elsewhere). A common shape:
  ~₹149/month, ~₹999/year (yearly framed as "best value").
- Activate the subscription.

**2. Wire the code** (mirrors the IAP flow, but subs):
- `queryProductDetails` for `SUBS` → present base plans on the paywall.
- `launchBillingFlow` with the chosen base plan's offer token.
- `onPurchasesUpdated` → **acknowledge** → `Subscription.setActive(true)` (persist).
- On launch, restore via `queryPurchases(SUBS)` so active subs re-grant.
- Handle states: active, in grace period, on hold, cancelled (still active until
  expiry). Gate Pro features on "is currently entitled", not "purchased once".

**3. Test without being charged:**
- Play Console → Setup → **License testing** → add your Gmail.
- Upload to an **Internal testing** track, install via the opt-in link, buy with a
  test card (no real charge). Subscriptions run on accelerated test renewals.

## Paywall UX

- A clear "Go Pro" entry (drawer + settings). The paywall lists concrete benefits,
  shows both plans with the yearly one as best value, and a "Not now" escape.
- Never nag mid-task. Offer Pro at natural moments (after using a Pro-gated feature),
  not on a blocking interstitial.

## Stubs are fine to ship behind a flag

It's OK to ship with billing in a safe stub (DEBUG-simulate purchase) and real ad
ids, then flip subscriptions on once the Play products exist. Just make sure the
DEBUG-only simulate affordances are `BuildConfig.DEBUG`-gated so they never reach
production.
