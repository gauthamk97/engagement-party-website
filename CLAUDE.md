# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static, no-build three-page website for Bhavna & Gautham's engagement party (home page, event details/menu, gift registry). Plain HTML/CSS/vanilla JS only — no package.json, no bundler, no framework, no templating system.

## Commands

There is no build, lint, or test tooling. To preview changes locally, serve the directory root and open it in a browser:

```bash
ruby -run -e httpd . -p 8791
```

Then visit `http://localhost:8791/index.html` (and `/details.html`, `/registry.html`). Use `ruby`'s built-in server specifically because this environment's `python3`/`node` are typically unavailable (missing Xcode Command Line Tools) — `ruby -run -e httpd` works without any extra install. Kill it with `pkill -f "ruby -run -e httpd"` when done.

Locally the `.html` extension is required in URLs. On the deployed site (GitHub Pages) it is not — see Routing below.

## Deployment

- Hosted on GitHub Pages from the `main` branch of `gauthamk97/engagement-party-website`, root directory.
- Custom domain `bhavnaandgautham.com` is wired via the `CNAME` file at the repo root (GitHub manages/recreates this file when the custom domain is set via the Pages API — don't be surprised by auto-generated commits touching it).
- Deploys automatically on push to `main`, typically live within ~1-2 minutes.

### Routing (important, non-obvious)

All internal links use extensionless paths (`/`, `details`, `registry`), **not** `index.html`/`details.html`/`registry.html`. GitHub Pages' static file server automatically resolves `/details` → `details.html` and `/registry` → `registry.html` with no redirect and no build step — this is native GitHub Pages behavior, not something configured in this repo. Keep writing links without `.html`; don't "fix" them by adding the extension back.

## Architecture

### No templating — shared markup is duplicated per page

There are exactly three pages: `index.html`, `details.html`, `registry.html`. Each is a fully self-contained HTML document with its own `<head>` (title, description, Open Graph/Twitter Card meta tags, favicon) and an identical copy of the header (`.site-header` / `.nav-wrap` / `.logo` / `.site-nav` / `.nav-toggle`) and footer (`.site-footer`) markup.

**When editing nav, footer, or any other shared chrome, the same change must be made in all three HTML files by hand** — there is no include/partial mechanism. The active nav link is marked per-page with `class="nav-link active"`.

### Shared assets

- `css/style.css` — single stylesheet for all three pages (~550 lines, no preprocessor).
- `js/main.js` — the only JS on the site; just toggles `.site-nav`'s `open` class from the mobile hamburger button (`.nav-toggle`). No other interactivity anywhere.
- `assets/` — real photos, pre-generated Venmo/Zelle QR code screenshots (`gautham-venmo.png`, `bhavna-zelle.png`, etc.), and `preview-image.png`, a branded wordmark image used as the `og:image`/`twitter:image` for link-preview cards (not a photo).

### CSS conventions

- Color palette lives entirely in `:root` custom properties: `--taupe` (headings, logo, primary buttons — the site's main accent), `--gold`/`--gold-light` (hover states, eyebrow text, GF diet tags), `--charcoal`/`--charcoal-soft` (body text, near-black/muted gray), `--sage`/`--sage-light` (Veg/Vegan diet tags), `--blush`/`--terracotta` (secondary accents). Change the palette by editing these variables, not individual rules.
- Fonts (Google Fonts, loaded via `@import` at the top of `style.css`): Playfair Display for all headings (`h1`–`h4`, serif), Cormorant Garamond italic for small accent/eyebrow text and menu sub-headings, Jost for body copy/nav/buttons.
- Global heading rule (`h1, h2, h3, h4 { color: var(--taupe); ... }`) sets heading color site-wide — don't add per-element color overrides for headings unless intentionally deviating from the global accent.
- A `.footnote` / `.menu-note` pattern (smaller, italic, muted) is used throughout for de-emphasized asides (e.g. the no-tie dress-code note, the vegan/GF legend). When scoping a new footnote-style rule, nest it under its container (e.g. `.detail-content .footnote`) rather than using a bare class selector — a bare single-class selector loses a CSS specificity fight against the existing `.detail-content p` / `.menu-card p` rules (one class vs. class+element), so the override silently fails.

### Page-specific structure

- **`index.html`**: `.hero` (name, date/time/after-party card, RSVP button, proposal photo) + `.welcome` (second proposal photo + intro copy). No details/menu content lives here — that's all on the Details page.
- **`details.html`**: `.page-intro` + `.details-list` (a 2-column/3-row grid of `.detail-row` items — Date & Time, Location, Getting There, Dress Code, Food & Drinks, Gifts — styled as text separated by divider lines, deliberately *not* card tiles) + `.menu-section` (two `.menu-card`s: Food, with categorized `.menu-list` items and V/VEG/GF `.diet-tag` pills; Drinks, with signature cocktails/open bar copy).
  - The `.detail-row` border-bottom logic is intentionally responsive: on the 2-column desktop grid only the last *row* (last 2 DOM items, via `:nth-last-child(-n+2)`) loses its divider; a `@media (max-width: 860px)` override restores the divider on the now-visually-mid-list second-to-last item once the grid stacks to 1 column. If you reorder or add/remove `.detail-row` items, re-check this still lands on a full last row (even item count) or the border logic will look wrong.
- **`registry.html`**: `.page-intro` + `.registry-grid` (two `.registry-card`s — "Our First Home Fund", "Honeymoon to Egypt Fund" — each with a cover photo and a `.payment-methods` block containing side-by-side Venmo (QR image + link button) and Zelle (QR image + phone number) `.payment-method`s).

### Responsive breakpoints

Two breakpoints, used consistently: `860px` (all major grids — hero, welcome, registry, menu, details-list — collapse from 2 columns to 1) and `640px` (nav collapses into the hamburger menu, spacing tightens).
