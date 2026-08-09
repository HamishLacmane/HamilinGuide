# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static, multi-page site (no framework, no build step, no package.json) — `index.html`, `food/index.html`, `travel/index.html`, sharing one `styles.css` at the repo root. Deploys to GitHub Pages with zero configuration. To preview locally, open `index.html` in a browser, or serve the folder with any static file server (relative links like `/food/` need a server, not `file://`, to resolve correctly).

## Fixed Michelin palette — do not add a switcher

`styles.css` defines one fixed set of tokens (`--paper: #FFFFFF`, `--ink: #191919`, `--accent: #BA0B2F`, plus derived greys) matching Michelin's real palette, pulled live from guide.michelin.com during design. This is intentional and permanent: Hamilin Guide is a deliberate Michelin Guide parody. Unlike Hamilin Hub's multi-palette switcher, Guide does **not** get a palette switcher — don't add one or suggest swapping the accent colour off Michelin red.

## Content is all placeholder — do not write copy

Every headline, intro paragraph, and guide entry across all three pages is either literal "placeholder text" or an empty `<!-- TODO(Hamish): ... -->` comment. This is deliberate, not an oversight — Hamish writes his own site copy and has said so explicitly more than once. Do not fill these in with invented restaurant names, descriptions, or headlines, even as "example" content — leave the TODO markers and placeholder structure for him to replace.

## Licence

No LICENSE file by design — all rights reserved by default. Don't add an open-source license without being explicitly asked to change that.
