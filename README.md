# Hamilin Guide

A personal, Michelin-styled guide — part of the Hamilin brand family. Static site, no framework, no build step, deploys to GitHub Pages with zero configuration.

## Structure

- `/` — the actual restaurant guide: a Michelin-style card grid with filters, grouped into sections (Top Picks, The Gelato Guru's Faves, Gone But Not Forgotten, All), reading `data/guide.sample.json`. Cards expand into a modal showing the full per-dish tier breakdown.
- `/travel/` — coming soon

## Data

`data/guide.sample.json` is currently a small **hand-picked sample** (3 real restaurants, real dish/tier data pulled from Hamilin Star's Postgres, manually tagged into a category each) — not a live production export. `hamilin-star-db/export_guide.sql` has the real query for the one category with defined criteria so far ("Gone But Not Forgotten" = permanently closed restaurants); the other categories' criteria are still undecided. See `CLAUDE.md` for the full picture.

## Palette

Unlike Hamilin Hub (which has a switchable multi-palette system), Guide's colours are **fixed** to Michelin's own — white ground, near-black ink, `#BA0B2F` red accent — defined once in `styles.css`. This is deliberate: Guide is a tongue-in-cheek parody of the Michelin Guide and wears their palette on purpose, regardless of whatever direction the rest of the Hamilin family ends up using. Each guide category also gets its own accent colour — crimson for Top Picks, pistachio green for The Gelato Guru's Faves, muted grey for Gone But Not Forgotten — applied to the category badge inside the expanded modal card; the grid card itself stays a plain uncoloured hairline border.

## Status

Site structure and interactivity are real and working (filtering, the expandable card modal, category colours). Headline, intro, and comment copy is left as literal placeholder text / `TODO(Hamish)` comments — intentionally not written by Claude, see `CLAUDE.md`.

## Licence

© 2026 Hamish Lacmane. All rights reserved. This code is provided for reference only and may not be copied, reused, or redistributed without permission.
