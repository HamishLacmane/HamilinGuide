# AGENTS.md — HamilinGuide

Two project agents run the Hamilin data flow. Their prompts are versioned here in `.claude/agents/` (kept identical in HamilinStar and HamilinGuide; the data pipeline they operate on lives in `../hamilin-star-db`, which is not a git repo).

| Agent | Prompt | Does |
|---|---|---|
| **hamilin-import-agent** | `.claude/agents/hamilin-import-agent.md` | Takes visits as Hamish gives them, resolves them against live Postgres, asks about doubts **one question at a time**, writes + applies guarded SQL to `manual_inserts.sql`, verifies, logs. |
| **hamilin-export-agent** | `.claude/agents/hamilin-export-agent.md` | Regenerates `data/restaurants.json`, `data/stars.json` (HamilinStar) and `data/guide.json` (HamilinGuide) from Postgres via a validated temp-dir swap, then reports `git diff --stat`. |

Flow: visits → import agent → Postgres → export agent → `data/*.json` in HamilinStar and HamilinGuide → Hamish previews locally → commit + push (feature branch, **no `Co-Authored-By` trailer**).

Shared rules: **temp files only inside the current project folder (deleted afterwards), never `/tmp`/AppData/scratchpad**; `PROJECT_LOG.md` is **append-only**; Postgres is the source of truth for new data (Airtable's free tier blocks new records); a `todo:`-prefixed message means log only; UK English.

**Keeping the copies in sync:** the same two prompt files exist in HamilinStar, HamilinGuide and the working copy at `Claude_code/.claude/agents/` (not under git). After changing one, copy it to the others so they don't drift, and add a line to each repo's `PROJECT_LOG.md`.

## Pre-push visual check (standing rule, added 2026-11-19)

**Before ANY commit + push of HTML/CSS/JS changes in this repo, verify the site looks right at several screen sizes. Do not push if any check fails; fix it first.** A data-only change (JSON/log) still needs the quick check on the affected page. The reason: a 60ch max-width left the Star intro wrapping at half the page width for weeks, and nobody saw it because it was only ever looked at in one window size.

**How:** start the local server with `preview_start` (config name below, never `python -m http.server` via Bash), then for every page in this repo's list and every size in the table use the Browser pane: `resize_window` (custom width x height; reset with `preset: "desktop"` at the end), reload (the local server caches hard: add `?cb=<n>` to the URL and relink the stylesheet, or `fetch(..., {cache:'no-store'})`), then check.

| Size | Width x height | Stands for |
|---|---|---|
| Small phone | 360 x 740 | narrow Android |
| Phone | 375 x 812 | iPhone-class (the `mobile` preset) |
| Tablet | 768 x 1024 | tablet portrait (the `tablet` preset) |
| Small laptop | 1024 x 768 | tablet landscape / small laptop |
| Laptop | 1280 x 800 | common laptop |
| Desktop | 1440 x 900 | desktop |
| Wide | 1920 x 1080 | large monitor |

**Checks at every size:**
1. **No horizontal page scroll.** Wide tables may scroll *inside* their own wrapper, but the page itself must not (`scrollWidth <= clientWidth`). Run the snippet below.
2. **No console errors** (`read_console_messages`, errors only) and no failed data fetches.
3. **Text uses the available width.** Paragraphs must not wrap far short of their container (watch for stray `max-width` caps such as `60ch`); long text must not overflow its box either.
4. **Nav, filter bars and button rows wrap cleanly** — nothing cut off, overlapping, or pushed off-screen; controls remain tappable (not tiny) on phone sizes.
5. **Tables and cards:** header and first columns readable, nothing clipped, badges/pills not overlapping text.
6. **Modals/popups** (open one on Guide): fit inside the viewport, scroll inside themselves, close button reachable, no content cut off.
7. **Look at it.** Take a screenshot at phone, tablet and desktop for every changed page and actually inspect it. DOM numbers alone miss ugly layouts.
8. **New or changed pages are checked in full; unchanged pages get at least phone + desktop.**
9. **Diff sanity before pushing:** `git diff --stat` must match what you changed. A 1-line edit showing ~1,000 changed lines means line endings were converted (CRLF): fix it, don't push (see the 2026-11-19 `PROJECT_LOG.md` entry).

**Overflow snippet** (paste into `javascript_tool`; `overflowing` should be empty and `hScroll` false):
```js
(() => {
  const vw = document.documentElement.clientWidth;
  const bad = [...document.querySelectorAll('body *')].filter(e => {
    const r = e.getBoundingClientRect();
    return r.width && (r.right > vw + 1 || r.left < -1) &&
      !e.closest('.table-wrap, .msel-panel, [hidden], .guide-modal-backdrop:not(.open)');
  }).slice(0, 10).map(e => e.tagName + '.' + e.className + ' right=' + Math.round(e.getBoundingClientRect().right));
  return { vw, scrollW: document.documentElement.scrollWidth, hScroll: document.documentElement.scrollWidth > vw + 1, overflowing: bad };
})()
```

**Report in the push message** a short table: page x size = pass/fail (and what was fixed). If the Browser pane is unavailable, say so plainly and do not claim the check was done. Leave the preview servers running afterwards (Hamish reviews the same local pages), but reset the viewport emulation to `desktop`.

**This repo:** preview config `hamilinguide-local` (port 8124). Pages to check: / (index.html), with a card modal open; /about/; /snacks/, with a card modal open (use temporary sample categories if the data is empty, then restore it).
