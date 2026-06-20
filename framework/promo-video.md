# Promo video

A short vertical (1080×1920) promo/ad video for the store listing, social, and ads —
built from the launch kit you already have. Same **navy canvas + one gold accent** house
style as the app. ~16s, H.264 + AAC, ~2 MB.

> Reference build: **Gps Camera Location** — 7 scenes, 16.4s, drifting gold-bokeh +
> vignette decorations, AI background music. The repeatable pipeline is packaged as the
> `promo-video` skill (`.claude/skills/promo-video/`) with three scripts you run in order.

## When to make one

After launch, once you have framed store screenshots (see [launch-checklist](launch-checklist.md)
§2). The video reuses those screenshots as scenes — no new capture needed.

## The pipeline (4 steps)

All in a scratch dir (`/tmp/ad/`); scripts live in the skill's `scripts/`.

1. **Title + CTA cards** — `make_cards.py` (PIL): navy-gradient `00-title.png` (logo,
   app name in gold, tagline, gold feature bullets) and `99-cta.png` (DOWNLOAD FREE +
   Play badge). Keep card text ASCII-safe (`•`, not `✓` — tofu in Arial).
2. **Decorations** — `make_decorations.py` (PIL): `bokeh.png` (soft gold particles,
   1080×2300 so it drifts) + `vignette.png` (soft dark edges). **Subtle is the point** —
   bokeh alpha ~0.55. Heavy particles fight the screenshots.
3. **Music** (optional) — Higgsfield MCP `generate_audio`, model `sonilo_music`,
   no vocals, `duration` ≈ video length. It's **async**: the call returns `pending` →
   poll `job_display({id})` → `curl` the `rawUrl` `.m4a`. Preflight with `get_cost:true`
   (~1 credit). Any royalty-free track works too.
4. **Render** — `render.sh` (ffmpeg, two passes):
   - **Pass A** `base.mp4`: stills `-loop 1 -t 2.8` each, `scale=…:increase,crop=1080:1920,
     setsar=1,fps=30`, chained with `xfade` (fade/slideleft, dur 0.6).
   - **Pass B** final: overlay drifting bokeh → vignette → mux music (`afade` in/out,
     `-shortest`), AAC 192k. Output `~/Desktop/<app>-promo.mp4`.

## Scene recipe

Lead with the promise, end with the ask. 5–8 scenes, ~2.2–2.8s each:

`title card → home/dashboard → hero feature → secondary feature → map/proof → Pro/upsell → CTA`

## Gotchas (from shipping)

- **Music is async** — poll + download; the first response has no file.
- **No letterboxing** — `force_original_aspect_ratio=increase` + `crop`, never `decrease`.
  All scenes must share exact WxH/SAR/fps or `xfade` errors.
- **Subtle decorations** — this is a utility-app ad, not a music video.
- **`-shortest`** so the (slightly longer) music doesn't add a black tail.
- **Use the store screenshots** as scenes — they're already framed + captioned.

See the skill for runnable scripts and exact commands.
