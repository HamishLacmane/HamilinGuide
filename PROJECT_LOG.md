# PROJECT_LOG — HamilinGuide

**Append-only.** Add new entries at the bottom; never edit or delete past lines (if something turns out wrong, add a new line saying so). Record key changes, decisions, gotchas, and things Hamish asked for. Format: `- YYYY-MM-DD — [change|request|decision|bug|todo] text`.

`CLAUDE.md` explains how the site works; this file records *what happened and when*. Data refreshes come from `../hamilin-star-db` (see its own log for the batch detail). Site copy is Hamish's own voice — don't write it for him.

_Entries up to 2026-10-10 were backfilled from memory notes and CLAUDE.md history — dates are the ones recorded there._

## Log

- 2026-08-09 — [decision] Deliberate Michelin Guide parody; fixed Michelin palette (white / #191919 / #BA0B2F), no palette switcher.
- 2026-08-13 — [change] Real build: card grid, modal, filters (country/city/cuisine + min stars). Category colour belongs on the modal badge, not the card border. 📕 favicon + SEO.
- 2026-08-14 — [change] Real data pipeline (`data/guide.json` from `export_guide.sql`). Sections show 5 cards with "Show all". Hamish's intro copy supplied. About page added.
- 2026-08-18 — [request] Added Amazing Value and Overrated categories; Overrated hidden on the homepage and excluded from "All". Per-restaurant Tabler icons rendered on card and modal.
- 2026-08-19 — [decision] Gelato Guru colour confirmed final at #03cafc (known ~1.9:1 contrast, accepted). "Buy me a gelato" link added; About-page link added under the intro in accent red.
- 2026-08-27 — [request] Support link switched from PayPal.me to buymeacoffee.com/hamilingui6.
- 2026-08-30 — [request] Country/City/Cuisine filters made multi-select; Min stars toggle given the same chevron (red to match its border).
- 2026-08-30 — [request] Guide Comment feature: markdown-lite comments rendered in the modal. Comment moved above the ratings; every category shown as a badge (opened-from section first); cuisine tags dropped from the location line; map-pin icon on "View on Google Maps".
- 2026-08-30 — [bug] Modal always showed the alphabetically-first category regardless of which card was clicked; fixed by passing the section key through.
- 2026-08-30 — [todo] Open: placeholder comment text looks identical to a real comment. Awaiting Hamish's decision.
- 2026-09-19 — [change] Data re-export for the 2–19 Sep visits batch.
- 2026-10-01 — [change] Data re-export for the 23–30 Sep visits batch.
- 2026-10-10 — [request] Hamish asked for an append-only PROJECT_LOG.md per repo. Created this file.
- 2026-10-10 — [bug] Four commits (2026-09-19 and 2026-10-01 data-refresh PRs, both repos) carried a `Co-Authored-By: Claude` trailer, against Hamish's standing no-trailer rule for Hamilin repos (a harness attribution reminder arrived and should have yielded to that rule). Already merged; history not rewritten. No trailer on Hamilin commits going forward.
- 2026-10-11 — [change] Data re-export for the October visits batch (3/7/9 Oct): Suraj Sweet Mart new/updated dishes (Pane Cunzato has no Guide category so isn't in guide.json). Not yet committed.
- 2026-10-11 — [request] Hamish asked for the agents to be stored in git with their projects. Added `.claude/agents/hamilin-import-agent.md`, `.claude/agents/hamilin-export-agent.md` and `AGENTS.md` (identical copies in HamilinStar and HamilinGuide). Not yet committed.
- 2026-10-25 — [change] Import agent prompt updated (usual-orders shorthand); copy kept identical with the other repo and the working copy. Not yet committed.
- 2026-10-25 — [change] Agent prompts + AGENTS.md updated: temp files only inside the current project folder (Hamish's rule). Copies kept identical. Not yet committed.
- 2026-10-25 — [change] Re-exported data/guide.json (70 restaurants, was 69; Pane Cunzato now has a Guide category) after the 14-24 Oct 2026 visits batch. Not yet committed.
- 2026-11-01 — [change] Re-ran guide export after the Nando's Cabot Place visits (28 and 29 Oct 2026, 4 reconfirmed dishes); guide.json 70 records, output byte-identical to the committed data (no exported field changed). Not yet committed.
