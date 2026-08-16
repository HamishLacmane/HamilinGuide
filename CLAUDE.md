# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static site (no framework, no build step, no package.json) — `index.html` is now the actual restaurant guide (a Michelin-styled card grid, replacing the old "landing page with two tiles" placeholder), `travel/index.html` is a standalone coming-soon stub, sharing one `styles.css` at the repo root. Deploys to GitHub Pages with zero configuration. To preview locally, serve the folder with any static file server (`python -m http.server` — relative fetch of `data/guide.sample.json` needs a server, not `file://`, to resolve).

**The old `/food/` page no longer exists** — its content is fully superseded by `index.html` now being the restaurant view directly (matching the real Michelin Guide, where the homepage *is* the restaurant list, not a link to it). Don't recreate a `/food/` route.

## Design — modelled on a real Michelin Guide screenshot, iterated with Hamish over several rounds on 2026-08-13

- **Nav**: `GUIDE` wordmark, pill-style tabs (`Restaurants` active, `Hotels`/`Travel guides` greyed "coming soon" — both plain unclickable `<span>`s, no page exists for either; `/travel/` was removed 2026-08-13, travel content is TBD), plus a clearly-labelled outbound link "See every rating on Hamilin Star ↗" so it's obvious that leads to the full/unfiltered data on the sister site, not more Guide content.
- **Section expand/collapse (added 2026-08-14)**: each section shows at most 5 cards by default, with a "Show all N ↓ / Show fewer" toggle (full-width row, `grid-column: 1 / -1`) once a section has more. State is per-section and survives filter changes.
- **Intro text box** directly under the nav — placeholder only, Hamish writes his own (see "Content is all placeholder" below).
- **Filter bar**: Country / City / Cuisine dropdowns (populated from whatever's in the loaded data) plus a "Min stars ▾" toggle that reveals four number inputs — minimum gold / silver / bronze / total. These are floors ("at least N"), not exact matches, by explicit request.
- **Sections, in this order, title case** (deliberately capitalizing small words like "But" too — not standard title-case rules, Hamish asked for it explicitly): **Top Picks**, **The Gelato Guru's Faves**, **Gone But Not Forgotten (Deprecated)** — "Deprecated" is a parenthetical qualifier on this section, not its own section — and **All**, which is intentionally just a "Coming soon" placeholder, not wired to real data yet. Every section heading is bigger than body text with a one-line description underneath (also placeholder).
- **Cards**: no heart/like icon (explicitly not wanted). No live map preview embed on the card or in the expanded view — GitHub Pages has no way to do that without either a paid API key or committing per-restaurant screenshot images to the repo, so it's a plain "View on Google Maps ↗" outbound link instead. Photo areas are optional placeholders (`no photo yet`) — Hamish is undecided whether he'll ever add real images, so the layout must work with or without them.
- **Star badge**: instead of Michelin's clover icons, a small colored circle (gold `#EF9F27` / silver `#9CA3AF` / bronze `#C97C42`, tokens in `styles.css`) containing a star glyph, with a "Tier ×N" count label — shows the restaurant's *dominant* tier (highest tier with at least one dish) and how many dishes hold it. A fourth "avoid" tier in crimson was discussed as a stretch idea but is **not implemented** — flagged as a possible future addition, not built.
- **Expanded/modal card** (not a separate page, deliberately light on GitHub Pages): photo placeholder, name, location + cuisine, an open/closed status line, then **which guide category the restaurant belongs to** as a pill (e.g. "Gone But Not Forgotten") placed *above* the dish list and *below* the status line — grouped with the other structural metadata, not down with the personal tags. **The per-category accent color (`--cat-top_picks`/`--cat-gelato_guru`/`--cat-gone_but_not_forgotten`) lives on this modal pill only** — the grid card itself (`.rcard`) stays a plain uncolored hairline border; Hamish was explicit that the card shouldn't change color, only this badge (corrected 2026-08-13 after the initial build wrongly colored the card's left border instead). Then the **full per-dish tier breakdown**, one row per dish in "Tier — Dish name" format (e.g. "Gold — Spicy Chicken Fillet Burger"), each row background-tinted to match its tier. Then Hamish's own comment (placeholder), a couple of tags (placeholder), the Google Maps link, and a link to `https://hamilinstar.com/stars.html?restaurant=<name>` ("See all ratings for this restaurant on Hamilin Star ↗") — same query-param pattern Hamilin Star's own cards already use for their own detail links.

## Data — real export pipeline, wired 2026-08-14 (was a 3-restaurant hand-picked sample before this)

`data/guide.json` (was `data/guide.sample.json` — renamed, the sample is gone) is generated by `hamilin-star-db/export_guide.sql`, same manual "run the SQL, copy the JSON output into this repo" loop already established for HamilinStar (`export.sql`/`export_stars.sql`) — no live backend, this is a static site. Regenerate whenever the underlying Postgres data changes:
```
docker exec -i hamilin-star-db-db-1 psql -U hamilin -d hamilin_star -t -A < export_guide.sql > ../HamilinGuide/data/guide.json
```

**Categories come from a real Airtable field now** — `Guide Categories` (a `multipleSelects` field on Restaurants, deliberately named with the "Guide" prefix to avoid colliding with Hamlin Stars' own unrelated `Category` field), imported into Postgres as `restaurant_guide_categories`/`guide_categories` (same multi-select → lookup + junction table pattern as `cuisine_tags`). This **replaces** the old `restaurant_status = 'Permanently Closed'` CASE-WHEN hack — categories are now hand-curated by Hamish in Airtable, independent of a restaurant's open/closed status, and a restaurant can carry more than one category (e.g. a Gelato Guru's Faves pick can also be tagged `Desserts`).

**`Desserts` is a 4th real category with no rendered section yet** (added 2026-08-14, alongside `Top Picks`/`The Gelato Guru's Faves`/`Gone But Not Forgotten`). Restaurants tagged *only* `desserts` (no other category) are present in `guide.json` but won't visibly render anywhere on the page until a Desserts section is actually built into `index.html` — this is documented behaviour, not a bug. `index.html`'s `openModal()` picks the first category the site *does* have a label/section for (`CATEGORY_LABEL`), skipping `desserts` for badge display purposes, so a restaurant with both `desserts` and a real category still shows the real one rather than silently hiding its badge.

**`cuisine_tags` doubles as the modal's "tags" placeholder content** (decided 2026-08-14, reusing existing data rather than a new field) — real chips render when present, the literal placeholder chip shows otherwise. Note it also already appears in the location line (`modal.loc`), so it's intentionally shown twice.

**`comment` (Hamish's own "why this is here" blurb) has no real data source yet** — `export_guide.sql` always selects it as `NULL`; the modal keeps showing literal placeholder text until a dedicated Airtable field (`Guide Comment`, proposed name) exists and the export is updated to select it. See the "New todos" section of the [[project_hamilin_sites]] memory for the full list of restaurants currently needing this written.

**A restaurant only appears in this export once it has at least one Guide Category** — Guide is explicitly a curated subset of Star's full data, not a mirror of everything Star has. 28 restaurants qualify as of 2026-08-14.

## Content is all placeholder — do not write copy

Every headline, intro paragraph, guide entry description, comment, and tag across the site is either literal "placeholder text" or an empty `<!-- TODO(Hamish): ... -->` comment. This is deliberate, not an oversight — Hamish writes his own site copy and has said so explicitly more than once. Do not fill these in with invented descriptions, comments, or headlines, even as "example" content — leave the TODO markers and placeholder structure for him to replace. The restaurant *names* and *dish/tier data* in `data/guide.sample.json` are real (pulled from Postgres) — the placeholder rule applies to Hamish's own voice/opinion content (intro copy, comments, tags), not to factual data.

## Fixed Michelin palette — do not add a switcher

`styles.css` defines one fixed set of tokens (`--paper: #FFFFFF`, `--ink: #191919`, `--accent: #BA0B2F`, plus derived greys and the gold/silver/bronze/no-star tier tokens added 2026-08-13) matching Michelin's real palette, pulled live from guide.michelin.com during design. This is intentional and permanent: Hamilin Guide is a deliberate Michelin Guide parody. Unlike Hamilin Hub's multi-palette switcher, Guide does **not** get a palette switcher — don't add one or suggest swapping the accent colour off Michelin red.

## Verification workflow

Before pushing any HTML/CSS/JS change, serve the repo locally (`python -m http.server <port>` from the repo root, or the `hamilinguide-local` launch config) and check it in the Browser tool — `read_console_messages` for JS errors, `javascript_tool` to inspect rendered DOM, click cards, and toggle filters, not just eyeballing. Same approach already established for HamilinStar.

## Still to do (as of 2026-08-14)

- **Categories are now real, curated in Airtable** (`Guide Categories` field → `restaurant_guide_categories`, see "Data" above) — `top_picks`/`gelato_guru`/`gone_but_not_forgotten` all have real tagged restaurants now (28-30 as of 2026-08-14), no criteria-definition blocker left for those three specifically. **`Desserts` is a 4th real category with no rendered section yet** — tagged restaurants exist in `guide.json` but are invisible until a Desserts section is built.
- An About page (requested 2026-08-13, not started).
- Build the "All" section for real (currently a static "Coming soon").
- `Guide Comment` field (Hamish's per-restaurant blurb) doesn't exist in Airtable yet — `export_guide.sql` always selects `comment` as `NULL`. Once it exists, Hamish still needs to write the actual text for all qualifying restaurants (explicitly his to write, not Claude's).
- Decide whether to ever commit real photos, and whether/how to handle the "avoid" red tier idea.
- New Guide Category idea floated 2026-08-14, **not started, don't build speculatively**: "Value for money" (naming undecided — leaning "Amazing Value" as of last discussion, not finalized). Wait for Hamish to supply qualifying restaurants and confirm the name before creating the Airtable option.
- Favicon and SEO pass — **done 2026-08-13**: favicon is 📕 (closed red book, a nod to Michelin's own "Red Guide" nickname — Hamish is undecided vs. 📖 open book, may switch later); meta description/OG/Twitter Card tags, canonical link, and a visually-hidden `<h1>` all added, mirroring Star's existing pass. Hub/Cards/Lacmane still need their own favicon + SEO work.
- **Icon-per-restaurant system — data layer BUILT 2026-08-16, not yet rendered anywhere on this site.** A nullable `icon` text column now exists on `restaurants` in `hamilin-star-db`, populated by rule (`ice-cream-2` for Gelato Guru-tagged places, plus name-based overrides for the rest — `burger`/`cookie`/`grill`/`soup`/`cup`/`dumpling`/`fish`/`carrot`) in `build_import.py`. **Still to do:** `export_guide.sql` doesn't select `icon` yet, so it isn't in `data/guide.json`; once it is, still needs an actual UI treatment here (e.g. shown on the grid card or in the modal) — not designed yet. See `hamilin-star-db/CLAUDE.md` and [[project_hamilin_sites]] for the full rule list and verified counts.
- **Guide's "No Star" dish-row modal treatment — DONE 2026-08-16.** `.dish-row.no-star` in `styles.css` (transparent background, `--hairline` border, `--ink-soft` text — Hamish's preferred "not endorsed, not another tier colour" direction) applied via a tier-check in `openModal()`'s dish-mapping in `index.html`, replacing the shared `TIER_STYLE` tint/ink lookup for this one tier. Verified live via computed-style check.

## Licence

No LICENSE file by design — all rights reserved by default. Don't add an open-source license without being explicitly asked to change that.

## Git workflow

`gh` CLI is not installed on this machine — PRs can't be created programmatically. Push the feature branch and hand Hamish the GitHub "create PR" URL GitHub prints on push. After Hamish merges, check whether the branch still exists on origin before reusing it — he periodically deletes merged branches via the GitHub UI.
