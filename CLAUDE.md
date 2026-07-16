# Scripbox Case Study — Project Context

## What this is

An interactive, slide-based case study for the portfolio of Sushma Bhandari, Senior Product Designer. It tells the story of leading design for Scripbox, a wealth management platform (client portal redesign, done with the agency Parallel). Chaptered, keyboard-navigable presentation in the browser: one idea per slide, motion only where it carries meaning. Chapters 1–3 are built; chapters 4–5 may be added later, so build with extension in mind.

## Hard rules (from the original spec, still in force)

1. **Copy is locked.** Headlines and captions provided by Sushma are used verbatim, character for character. Do not improve, shorten, or rephrase. Only `[square bracket]` placeholders are editable.
2. **No em dashes or en dashes anywhere** — not in copy, UI labels, alt text, or code comments that render to screen.
3. **Do not invent metrics, features, or claims** not provided by Sushma.
4. **Single self-contained deliverable**: one `index.html` with embedded CSS and vanilla JS. No frameworks.
5. **Must work over `file://`** — Sushma opens the file by double-clicking, no local server. Never use `fetch()` for local assets (CORS blocks it); use `<img>` tags. Quote URLs in `background-image: url("...")` since asset paths contain spaces.

## Workflow conventions

- `index.html` is always the **live, latest** version — the one Sushma opens and reviews.
- `versions/` holds numbered backups (`v1.html` … `v8.html`). **After every meaningful round of changes, copy the result to the next `versions/vN.html`.** The folder is rollback history only, not for day-to-day review.
- **Verify changes visually** with headless Chrome screenshots before reporting done. The deck ignores the URL hash on load (always opens on the cover), so `index.html#slide-3` no longer works for screenshots; instead load the deck in an iframe wrapper and set `location.hash` (or dispatch ArrowRight keys) after load — the `hashchange` listener still navigates. Test at a MacBook-typical width (~1512px), not 2000px — a headline-wrap bug once slipped through because testing was at 2000px.
- **Uniform scale-to-fit responsive strategy (no reflow).** The whole deck (slides + all chrome) lives in `#stage`, laid out at a fixed 1512x950 design resolution — all sizes are px (former vw/vh values were resolved at 1512x950; never reintroduce viewport units inside the stage). `fitDeck()` applies one `transform:scale(...)` on load/resize/orientationchange (plus a 300ms delayed re-fit on orientationchange for iOS stale dims, and a `visualViewport` resize listener); leftover space letterboxes in `--bg`. It measures `document.documentElement.clientWidth/Height` (layout viewport), **never `window.innerWidth/Height`** — mobile browsers change the latter with pinch zoom, auto shrink-to-fit and the URL bar, which broke scaling on phones once. There is no separate mobile layout or breakpoint reflow. **Portrait on touch devices auto-rotates**: `fitDeck()` sets `deckRotated` and applies `rotate(90deg) scale(min(vh/1512, vw/950))` so the deck reads landscape without turning the phone (true orientation lock isn't possible on the web; Screen Orientation API needs fullscreen and iOS Safari lacks it). Portrait on non-touch still shows the "Rotate your device" pill (`#rotate-hint`, outside the stage). `touchDevice` is `(pointer: coarse)` only, so touchscreen laptops with a mouse keep desktop chrome. **Touch devices navigate by swipe** (touchstart/touchend on document, gesture mapped through `designDelta` so the axis follows the rotation; 48 visual px minimum so taps still click); `body.touch` hides the `#nav-arrows` buttons and the cover hint reads "Swipe to begin" (set in JS at init). `#stage` must stay `position:fixed` (as absolute, its oversized/rotated box made the document scrollable on phones), and a non-passive `touchmove` listener preventDefaults single-finger panning — iOS Safari pans on touch even with `overflow:hidden`. JS that measures with `getBoundingClientRect` (slide 2 merge, slide 8 arrows, slide 12 arrow) must go through the rotation-aware helpers `designRect(el, refRect)` / `designDelta(vdx, vdy)` next to `fitDeck()` — never divide raw rect deltas by `deckScale` directly, that breaks when `deckRotated` swaps the axes. The `dark-break` class is toggled on both `<body>` and `<html>` (html backs the safe-area/letterbox under `viewport-fit=cover`).
- Desktop notifications are configured in `~/.claude/settings.json` (Notification + Stop hooks); don't overwrite them.

## Design system

- **Palette** (`:root`): `--bg:#FAFAF7` (cream), `--ink:#141414`, `--muted:#6E6E66`, `--accent:#F58024` (Scripbox orange), `--line:#DDDDD4` (hairlines).
- **Never use blue accents.** `--accent` was deliberately changed from blue `#1F4FD8` to Scripbox orange — everything accent-colored is orange, deck-wide.
- **Fonts** (Google Fonts): Marcellus for all headings/headlines (`h1,h2,h3` and stat figures) — **always weight 400 and `letter-spacing: -0.46px`** (Marcellus ships in 400 only; never fake-bold headings). Replaced Libre Baskerville in the v10 round. Spline Sans Mono for eyebrows, tags, labels, tables. Inter for body, Archivo for small bold UI labels.
- **Split-slide layout language**: text on the left, visual on the right. Content-heavy slides use a filled right panel in soft peach (orange at ~12% tint, e.g. `#FDE6D6`) that bleeds edge-to-edge (top/right/bottom) — established on slides 1, 2, 4b, 8. Text-only statement slides stay centered/left at ~75% width.
- **Card language**: white cards, hairline `--line` border, ~10px radius, mono uppercase tags with a hairline divider under the tag.
- **Bottom chrome (all slides)**: sleek frosted capsule holding the carousel dots, centered at the bottom; "Contents" label revealed on hovering the capsule (click opens the contents overlay); circular ←/→ arrows at bottom-right (Sequifi style). The "Sushma Bhandari · Scripbox × Parallel" attribution strip and page counter are **hidden but kept in the file** — placement TBD by Sushma.
- Reduced-motion: body gets a `reduced-motion` class; animated sequences need static fallbacks.

## Style references (interaction paradigm only, not visual style)

- Sequifi deck: https://pavan-parallel.github.io/sales-enablement/sequifi-ds-v4.html (fan of screens, corner arrows, guided vertical list at `#12.2`)
- Healthify deck: https://pavan-parallel.github.io/sales-enablement/healthify.html (card grid → verdict statuses → re-sort beats)

## File structure

```
index.html              live deck (single file, ~2200 lines)
versions/v1..v18.html   version history (v14 missing: lost to an accidental overwrite;
                        always `ls versions/` for the real latest before saving a new one)
assets/
  logo orange/black/white.png, logo.svg
  india-map.svg         strokes burnt orange #B96A32; city coords calibrated to
                        NCERT extent (8°4'–37°6'N, 68°7'–97°25'E) — verified, don't eyeball
  scripbox-app-home.png slide 1 phone mockup, dot-grid bg extended by tiling (period 57px)
  page 3 CP/            5 client-portal screenshots for the slide 3 fan (~1512×982)
  report screens/       5 report PNGs (1920×1080) for slide 4b; note the filenames:
                        overall wealth / long term growth / downside protection /
                        index banchmarking (sic) / rebalancing
  old navigation.png, new navigation.gif   slide 8 IA assets
```

## Current slide map (cover + 15 dots + 4 dark break slides)

**Cover slide** (`data-type="cover"`, before the "Context" break, deck opens on it): headline "Growing Scripbox's desktop experience, from one segment to every user", subcopy, and the 3 impact stats (2.2x logins QoQ / +312% repeat users / 25% 30-day return, `.impact-stats` shared with dot 14). No dot, no page number, not in the contents overlay; name tag stays visible.

**The chapter system is gone** (removed in the v11 round): no `data-chapter` attributes, no per-slide eyebrows ("01 · The brief" style). Section breaks are 4 dark film-title cards (`.interstitial`, `data-type="interstitial"`, near-black `#141414` bg, centered Marcellus title in cream, no eyebrow/numbering, excluded from dots and contents): "Context" before dot 1, "Performance reporting" before dot 4, "Redesigning the invest flow" before dot 11 (the transactions checklist slide), "Impact" before dot 14, plus a fifth dark card "Thank you" as the very last slide in the deck (after dot 15; a trailing interstitial highlights no dot and starts no dot group). The contents overlay is one flat numbered list of all content slides. The dot capsule groups dots by break slide (gaps between groups; hover label shows the break title) — **groups are derived at runtime from deck order** (each interstitial's h2 starts a new group), so inserting slides or breaks needs no JS change. Every content slide shows a fixed mono page number top-left (`#page-num`, "06 / 15" style, driven by `updateChrome`, hidden on breaks).

1. **Hero** — locked headline "Scripbox is one of India's largest wealth management platforms." Hard-coded 3-line break (India's / wealth management / platforms.), headline font `clamp(1.8rem, 3.1vw, 3.4rem)` so wraps hold at all widths. Stat row of three equal columns (₹21,000 Cr+ AUM / 1 lakh+ families / 2,500+ cities): hairline above, Libre Baskerville orange figure, muted label. AUM count-up animation retained. Right: ~32%-wide peach panel with the phone mockup image (sized by panel width so the phone stays modest). No credit line, no caption.
2. **Acquisitions** — India map (1.3× size, no border/frame) on the peach right panel with all acquired companies called out at once via pins (logo orange `#F58024`) and leader-line callout cards; **one click** merges them all into the Scripbox logo (large orange one, aligned to India's north–south spine, shifted ~15.6% left of frame center). No zoom interaction, no "watch them merge" nudge. Left: headline at 41vw + three bullets (own systems / RMs worked differently / no unified product) with orange dot markers; no company names in subcopy.
3. **Client portal fan** — single headline spanning 3/4 width, `Reporting · Portfolio viewing · Transactions(strike)` line beneath. The 5 portal screenshots fan along the **bottom** edge (Sequifi style), natural aspect ratio, no cropping, no translucency; hover lifts 5.5rem, scales +6%, straightens, comes to front.
4. **(4a)** Statement slide — the monthly RM call / PDF report sentence, 75% width.
5. **(4b)** Report walkthrough — 35/65 split, peach panel right with 16:9 report screens. Guided vertical list under heading "The portfolio report contains": **cumulative reveal** (items never dim once shown), rail draws top-to-bottom (400ms) then next item fades in; 300ms opacity crossfade on the screen; next-arrow drives the 5 steps before leaving the slide, backward un-reveals. Item titles ~1.2rem with muted subcopy. After the 5th item, a mono chip "+ and many more" fades in as the closing beat.
6. **Problems** — "But none of it lived in the Client Portal." + three cards (PROBLEM 1/2/3 tags, divider, tight padding): Gated by an RM / Static reporting / No room to explore.
7. **Card sort** — "What should make it to the Client Portal?" 15 cards in a 5×3 grid, 5-beat sequence: (1) headline → RMs line, Healthify-style status pills appear in place — Keep=green (9), Remove=red (Inflation Benchmark, Market Drawdown, Fund Overlap), Upcoming version=yellow (Exit Recommendations, Rebalancing, SIP Actions); (2) red+yellow fall away, keepers revert to plain and **compact into the same 5×3 slots, left-aligned** (card x/y slots never re-center); (3–4) headline → organising reporting, cards glide into 4 groups (Home / Performance insights / Distribution / Holdings).
8. **Information architecture** — two-half layout, larger filled right panel (edge-to-edge like slides 1–2), browser-framed screen: `old navigation.png` → click → `new navigation.gif`. Left: "Problem 1/2/3" cards (old) / "Solution 1/2/3" (new) revealing cumulatively (cards stay visible once shown) with curved orange arrows pointing to component **edges**, not centers, never overlapping text. SVG overlay id is `ia-arrows` — **never reuse the deck's `nav-arrows` id** (doing so once hijacked the ←/→ buttons' position).
9. **Bigger question** — "The upgraded client portal raised a bigger question." + subcopy, comparison table (Client Portal vs Legacy Website: Audience / Reach 10%–90% / Experience) in mono style, then serif takeaway with orange accent bar: "Why should the better desktop experience be limited to just 10% of users?"
10–13. **The merge section** — one platform decision, transact capability, invest flow redesign, "We redesigned the invest flow around four key principles." (largely as originally built; not yet reworked in review rounds).
14. **Summary** — "How we transformed the desktop experience" + three `.problem-card` cards reusing the slide 6 treatment (tags Reporting / Platform / Invest flow, body copy only, no titles; `.slide.ssum .problem-copy` adds the missing title's top margin).
15. **Partnership close (last content slide; the dark "Thank you" card follows it)** — centered `closing-frame` treatment as a content slide: mono "Scripbox × Parallel", headline "The partnership grew", muted subcopy teasing the app case study. No link (old "[Back to portfolio]" placeholder removed; the `data-type="closing"` slide type no longer exists in the deck).

## Known pending items

- "Sushma Bhandari" name tag (`#name-tag`) is fixed top-right on content slides only, mirroring `#page-num` (hidden on break slides via `body.break-active`). The fuller attribution strip ("Sushma Bhandari · Scripbox × Parallel") remains hidden in the file.
- More content may come later; keep the deck extensible (dot renumbering ripples through the contents overlay, thumbs, and hardcoded dot numbers in the JS beat handlers — update all when inserting slides).

## Working with Sushma

- Prompts are often voice-dictated: expect filler words and transcription slips ("carousal", "Sequify", "Heldify"); read for intent.
- She reviews by refreshing `index.html` in her browser — remind her to hard-refresh (Cmd+Shift+R) if changes don't appear.
- Iterates visually in small rounds; when she shares a reference image or deck link, match the interaction/layout closely rather than interpreting loosely.
- Prefers subtle, sleek, editorial treatments: lighter weights, smaller headings, hairlines, generous whitespace, no jarring blacks (e.g., map strokes warmed to `#B96A32`).
