# AGENTS.md — HamilinGuide

Two project agents run the Hamilin data flow. Their prompts are versioned here in `.claude/agents/` (kept identical in HamilinStar and HamilinGuide; the data pipeline they operate on lives in `../hamilin-star-db`, which is not a git repo).

| Agent | Prompt | Does |
|---|---|---|
| **hamilin-import-agent** | `.claude/agents/hamilin-import-agent.md` | Takes visits as Hamish gives them, resolves them against live Postgres, asks about doubts **one question at a time**, writes + applies guarded SQL to `manual_inserts.sql`, verifies, logs. |
| **hamilin-export-agent** | `.claude/agents/hamilin-export-agent.md` | Regenerates `data/restaurants.json`, `data/stars.json` (HamilinStar) and `data/guide.json` (HamilinGuide) from Postgres via a validated temp-dir swap, then reports `git diff --stat`. |

Flow: visits → import agent → Postgres → export agent → `data/*.json` in HamilinStar and HamilinGuide → Hamish previews locally → commit + push (feature branch, **no `Co-Authored-By` trailer**).

Shared rules: **temp files only inside the current project folder (deleted afterwards), never `/tmp`/AppData/scratchpad**; `PROJECT_LOG.md` is **append-only**; Postgres is the source of truth for new data (Airtable's free tier blocks new records); a `todo:`-prefixed message means log only; UK English.

**Keeping the copies in sync:** the same two prompt files exist in HamilinStar, HamilinGuide and the working copy at `Claude_code/.claude/agents/` (not under git). After changing one, copy it to the others so they don't drift, and add a line to each repo's `PROJECT_LOG.md`.
