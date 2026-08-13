# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static site (no framework, no build step, no package.json) — `index.html` is now the actual restaurant guide (a Michelin-styled card grid, replacing the old "landing page with two tiles" placeholder), `travel/index.html` is a standalone coming-soon stub, sharing one `styles.css` at the repo root. Deploys to GitHub Pages with zero configuration. To preview locally, serve the folder with any static file server (`python -m http.server` — relative fetch of `data/guide.sample.json` needs a server, not `file://`, to resolve).

**The old `/food/` page no longer exists** — its content is fully superseded by `index.html` now being the restaurant view directly (matching the real Michelin Guide, where the homepage *is* the restaurant list, not a link to it). Don't recreate a `/food/` route.

## Design — modelled on a real Michelin Guide screenshot, iterated with Hamish over several rounds on 2026-08-13

- **Nav**: `GUIDE` wordmark, pill-style tabs (`Restaurants` active, `Hotels`/`Travel guides` greyed "coming soon" — Travel guides links to the real `/travel/` stub since that page exists, Hotels doesn't link anywhere since no page exists yet), plus a clearly-labelled outbound link "See every rating on Hamilin Star ↗" so it's obvious that leads to the full/unfiltered data on the sister site, not more Guide content.
- **Intro text box** directly under the nav — placeholder only, Hamish writes his own (see "Content is all placeholder" below).
- **Filter bar**: Country / City / Cuisine dropdowns (populated from whatever's in the loaded data) plus a "Min stars ▾" toggle that reveals four number inputs — minimum gold / silver / bronze / total. These are floors ("at least N"), not exact matches, by explicit request.
- **Sections, in this order, title case** (deliberately capitalizing small words like "But" too — not standard title-case rules, Hamish asked for it explicitly): **Top Picks**, **The Gelato Guru's Faves**, **Gone But Not Forgotten (Deprecated)** — "Deprecated" is a parenthetical qualifier on this section, not its own section — and **All**, which is intentionally just a "Coming soon" placeholder, not wired to real data yet. Every section heading is bigger than body text with a one-line description underneath (also placeholder).
- **Cards**: no heart/like icon (explicitly not wanted). No live map preview embed on the card or in the expanded view — GitHub Pages has no way to do that without either a paid API key or committing per-restaurant screenshot images to the repo, so it's a plain "View on Google Maps ↗" outbound link instead. Photo areas are optional placeholders (`no photo yet`) — Hamish is undecided whether he'll ever add real images, so the layout must work with or without them.
- **Star badge**: instead of Michelin's clover icons, a small colored circle (gold `#EF9F27` / silver `#9CA3AF` / bronze `#C97C42`, tokens in `styles.css`) containing a star glyph, with a "Tier ×N" count label — shows the restaurant's *dominant* tier (highest tier with at least one dish) and how many dishes hold it. A fourth "avoid" tier in crimson was discussed as a stretch idea but is **not implemented** — flagged as a possible future addition, not built.
- **Expanded/modal card** (not a separate page, deliberately light on GitHub Pages): photo placeholder, name, location + cuisine, an open/closed status line, then **which guide category the restaurant belongs to** as a pill (e.g. "Gone But Not Forgotten") placed *above* the dish list and *below* the status line — grouped with the other structural metadata, not down with the personal tags. **The per-category accent color (`--cat-top_picks`/`--cat-gelato_guru`/`--cat-gone_but_not_forgotten`) lives on this modal pill only** — the grid card itself (`.rcard`) stays a plain uncolored hairline border; Hamish was explicit that the card shouldn't change color, only this badge (corrected 2026-08-13 after the initial build wrongly colored the card's left border instead). Then the **full per-dish tier breakdown**, one row per dish in "Tier — Dish name" format (e.g. "Gold — Spicy Chicken Fillet Burger"), each row background-tinted to match its tier. Then Hamish's own comment (placeholder), a couple of tags (placeholder), the Google Maps link, and a link to `https://hamilinstar.com/stars.html?restaurant=<name>` ("See all ratings for this restaurant on Hamilin Star ↗") — same query-param pattern Hamilin Star's own cards already use for their own detail links.

## Data — currently a hand-picked sample, not a live category query

`data/guide.sample.json` has **3 real restaurants** (Amol's Vada Pav, Gelateria La Romana, OG Chicken Cottage (Alperton)) with real dish/tier data pulled from Hamilin Star's Postgres, each manually tagged into one category — this is deliberately a sample to validate the UI shape, not a production export. **Only `gone_but_not_forgotten` has defined selection criteria** (`restaurant_status = 'Permanently Closed'` in Postgres) — `top_picks` and `gelato_guru`'s real criteria are still undefined; Hamish hasn't said what qualifies a restaurant for either yet. Don't invent criteria for these — ask him when it's time to wire up the real export.

`hamilin-star-db/export_guide.sql` exists and correctly implements the `gone_but_not_forgotten` category for real (same flattened-JSON pattern as `export.sql`/`export_stars.sql`, but per-dish instead of aggregate counts, since the guide's expanded card needs the exact dish list) — **not yet run/wired into this repo's actual data file**, kept as the template for when the other categories' criteria are defined and the full pipeline gets built. When that happens, follow the same manual "run the SQL, copy the JSON output into this repo" loop already established for HamilinStar — no live backend, this is a static site.

**A restaurant only appears in this export once it has at least one non-empty category** — Guide is explicitly a curated subset of Star's full data, not a mirror of everything Star has.

## Content is all placeholder — do not write copy

Every headline, intro paragraph, guide entry description, comment, and tag across the site is either literal "placeholder text" or an empty `<!-- TODO(Hamish): ... -->` comment. This is deliberate, not an oversight — Hamish writes his own site copy and has said so explicitly more than once. Do not fill these in with invented descriptions, comments, or headlines, even as "example" content — leave the TODO markers and placeholder structure for him to replace. The restaurant *names* and *dish/tier data* in `data/guide.sample.json` are real (pulled from Postgres) — the placeholder rule applies to Hamish's own voice/opinion content (intro copy, comments, tags), not to factual data.

## Fixed Michelin palette — do not add a switcher

`styles.css` defines one fixed set of tokens (`--paper: #FFFFFF`, `--ink: #191919`, `--accent: #BA0B2F`, plus derived greys and the gold/silver/bronze/no-star tier tokens added 2026-08-13) matching Michelin's real palette, pulled live from guide.michelin.com during design. This is intentional and permanent: Hamilin Guide is a deliberate Michelin Guide parody. Unlike Hamilin Hub's multi-palette switcher, Guide does **not** get a palette switcher — don't add one or suggest swapping the accent colour off Michelin red.

## Verification workflow

Before pushing any HTML/CSS/JS change, serve the repo locally (`python -m http.server <port>` from the repo root, or the `hamilinguide-local` launch config) and check it in the Browser tool — `read_console_messages` for JS errors, `javascript_tool` to inspect rendered DOM, click cards, and toggle filters, not just eyeballing. Same approach already established for HamilinStar.

## Still to do (as of 2026-08-13)

- Define real criteria for `top_picks` and `gelato_guru` categories, then build the real `export_guide.sql`-driven data pipeline (replacing the hand-picked sample file).
- Build the "All" section for real (currently a static "Coming soon").
- An About page (requested 2026-08-13, not started).
- Decide whether to ever commit real photos, and whether/how to handle the "avoid" red tier idea.
- Favicon (Hub/Guide/Cards/Lacmane all still need one — HamilinStar already has its ⭐).
- SEO basics (HamilinStar already has a pass done — Guide doesn't yet).

## Licence

No LICENSE file by design — all rights reserved by default. Don't add an open-source license without being explicitly asked to change that.

## Git workflow

`gh` CLI is not installed on this machine — PRs can't be created programmatically. Push the feature branch and hand Hamish the GitHub "create PR" URL GitHub prints on push. After Hamish merges, check whether the branch still exists on origin before reusing it — he periodically deletes merged branches via the GitHub UI.
