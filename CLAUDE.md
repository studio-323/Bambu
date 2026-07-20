# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is the static marketing website for Bambu Philippines (bambuphilippines.com — see `CNAME`), a bamboo cladding/decking/flooring products company. It is plain HTML/CSS/JS with no build step, no package manager, and no test framework — every page is a hand-authored, self-contained `index.html`.

## Development

There is no build/lint/test tooling. To preview the site:

- Open any `index.html` directly in a browser, or
- Use the "Live Server" VS Code extension (`.vscode/settings.json` sets `liveServer.settings.port: 5501`, matching the repo's existing VS Code setup).

Deployment is via GitHub Pages (the `CNAME` file points the custom domain at the repo). Pushing to `main` is effectively "shipping" — there is no CI/staging step.

## Architecture

### Page structure — one folder per route

Every route is a directory containing its own `index.html` and `styles.css` (e.g. `About/`, `Contact/`, `FAQ/`, `Menu/`, `Portfolio/`, `Products/`, `Quotation/`, `Confirmation/`). Deeper routes nest further, e.g.:

```
Products/Product Pages/<category>/page<N>/index.html
Products/Product Pages/<category>/page<N>/styles.css
Products/Product Pages/<category>/page<N>/elements/   (page-local images)
Portfolio/portfolio pages/<favorites|others>/page<N>/index.html
```

Product categories currently present: `accessories`, `cladding`, `decking`, `flooring`.

### CSS is per-page, not shared

**Each page directory has its own independent `styles.css`.** These are *not* copies of a shared stylesheet — they diverge in content and line count per page. There is no global stylesheet import chain. When asked to change a style, first confirm which page(s) are affected and edit each page's own `styles.css` — a fix in `Products/styles.css` will not propagate to `Products/Product Pages/cladding/page1/styles.css` or any other page.

### JS is shared and global

`app.js` at the repo root is the single shared script, referenced from every page via a relative path (e.g. `../../../../app.js` from a deeply nested product page). It handles:

- `IntersectionObserver`-driven reveal animations for `.hidden` elements (adds/removes `.show`)
- The mobile hamburger menu toggle (`myFunction`, `#myLinks`)
- The homepage image slider (`showSlides`, `plusSlides`, `currentSlide`)
- The desktop nav dropdown toggle (`toggleMenu`, `.menu` / `.dropdown` / `header.black`)

Changes to `app.js` affect every page site-wide — check that selectors used (`.hidden`, `.menu`, `.dropdown`, `#myLinks`, `.mySlides`, `.dot`) still match a given page's markup before assuming a change is safe everywhere.

### Relative paths everywhere

There are no absolute paths or a templating system — every `<link>`, `<script>`, `<img>`, and nav `<a href>` uses a relative path computed from that file's depth (e.g. a page 4 levels deep references the root as `../../../../`). When copying markup between pages at different depths (a common source of bugs here), all relative paths must be re-counted for the new location, including:

- `styles.css` link
- `app.js` script
- `elements/favicon.png` favicon
- `Global Resources/bambu.png` logo
- Every nav link (Home / Products / Portfolio / Contact / FAQ) and the "Get a Quotation" Google Form link

### Assets

- Shared/global images live in `elements/` and `Global Resources/` at the repo root.
- Most page directories additionally have their own local `elements/` subfolder for page-specific images (numbered `element1.png`, `element2.png`, …).
- `elements/landing.mp4` is a large (~32MB) video asset used on the homepage — be mindful of this when doing bulk operations across the repo.

### `test/`

`test/index.html` and `test/styles.css` are a scratch/experimental page, not an automated test suite — there is no test runner in this project.
