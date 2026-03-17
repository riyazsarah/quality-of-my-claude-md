# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file HTML presentation for a Claude Code 201 training session. The presentation (`Claude_Code_201_Presentation.html`) is a self-contained slide deck with inline CSS, JS, and SVG assets. Speaker notes live in a separate Markdown file.

## Architecture

- **Presentation**: `Claude_Code_201_Presentation.html` — all slides, styles, animations, and navigation logic in one file
- **Assets**: `assets/` — crab mascot PNGs, GIFs, and SVGs referenced by the HTML
- **Speaker Notes**: `Claude_Code_201_Speaker_Notes.md` — per-slide talking points and demo scripts

### Slide System

- Slides use horizontal `translateX` scrolling on a `.slides-wrapper` div
- Each slide is a `<div class="slide">` with theme classes: `slide--dark`, `slide--darker`, `slide--warm`
- Navigation via `goTo(index)` / `navigate(dir)` JS functions
- Reveal slides (`data-reveal-slide` attribute) use click/keyboard to progressively show facts — `handleNextClick()` checks for reveals before navigating
- Slide numbers auto-sync via `syncSlideNumbers()` using `querySelectorAll('.slide').length`

### Key CSS Patterns

- CSS variables defined in `:root` (`--coral`, `--bg-dark`, `--bg-warm`, etc.)
- `.two-col` for two-column layouts, `.card-grid` for card grids
- `.code-block` for syntax-highlighted code (uses `white-space: pre-wrap`)
- `.mini-table` for data tables, `.hooks-ref` for the hooks reference table
- `.disclosure-card` / `.revealed-sidebar` for progressive reveal on slide 5

## Development

Open the HTML file directly in a browser — no build step required:

```bash
open Claude_Code_201_Presentation.html
# or serve locally:
python3 -m http.server 8765
```

## Conventions

- Always apply `/tufte-slide-design` principles when creating or modifying slides (maximize data-ink ratio, eliminate chartjunk)
- Use `trash` instead of `rm` for file deletion
- Prefer HTML components over external images when they need to render reliably across environments
- Navigation controls are positioned bottom-right (fixed)
