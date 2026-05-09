# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Static personal portfolio site (Anshul Jain — Unity Game Developer) deployed via GitHub Pages from the repo root (`shadowknight08.github.io`). Built on the **ikonik** vCard template by pixelwars (HTML5 + jQuery, no build system).

## Running locally

There is no build, lint, or test pipeline. Serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000
# or
npx serve .
```

`send-mail.php` (used by the contact form) requires a PHP-capable host and is **not** executed by GitHub Pages — the contact form's POST will fail in production unless the form action is repointed to a working endpoint (e.g., Formspree, Netlify Forms). Note the template-default `$to` and `$sender_domain` in [send-mail.php](send-mail.php) need to be replaced before that file is useful.

## Architecture

### Two layout modes, switched by an HTML class on `<html>`

[js/main.js](js/main.js) branches on which class is set on the root `<html>` element:

- **`one-page-layout`** ([index.html](index.html), `index2.html`, `index3.html`) — The home screen stays mounted; clicking a `.home-menu` link triggers an AJAX fetch of the target page (`about.html`, `resume.html`, `portfolio.html`, `contact.html`), injects its content into `.one-page-content > .content-wrap`, and updates the URL hash via `jquery-address`. Browser back/forward is wired through `$.address.change()`. Portfolio detail pages get a second-layer overlay (`.p-overlay`) loaded via the same AJAX path.
- **`classic-page-layout`** (e.g. [portfolio.html](portfolio.html)) — Standard full page navigation; the home/AJAX machinery is skipped.

When editing a section page, remember it must work **both** as a standalone document (classic load / direct URL) and when its body is grafted into the one-page shell. Don't add top-level `<html>`/`<head>` dependencies that the AJAX path won't carry over.

### Routing convention

`.home-menu a` elements declare their target via `data-slug` (e.g. `data-slug="about"` → loads `about.html`, URL becomes `#/about`). [js/main.js:56-63](js/main.js#L56-L63) rewrites the visible `href` to `#/<slug>` and stashes the real file URL in `data-file-url`. Adding a new top-level section means: (1) add the menu `<a>` with a `data-slug`, (2) create `<slug>.html` that mirrors the existing section structure.

### CSS and JS layering

- [css/normalize.css](css/normalize.css) → [css/bootstrap.css](css/bootstrap.css) (grid only) → [css/main.css](css/main.css) (theme) → [css/768.css](css/768.css) (mobile breakpoint overrides). Always preserve this load order in new pages — `768.css` must come last to win the cascade on mobile.
- jQuery is loaded in `<head>`; everything else (`tween-max`, `jquery-isotope`, `imagesloaded`, `magnific-popup`, `jquery-address`, `nprogress`, `jquery-validate`, `jquery-sticky-sidebar`, then `main.js`) is loaded at end of `<body>`. `main.js` assumes all of these are present.

### Image directories

- [images/](images/) — template-default assets (template authors' demo images, icons, fonts).
- [organized-images/](organized-images/) — the **real** portfolio content (`background-images/`, `portfolio-images/`, `site-images/`). Prefer this directory when adding/replacing portfolio assets. [organized-images/image-inventory.txt](organized-images/image-inventory.txt) catalogues what's there.

## Theme customization

[Customizer.txt](Customizer.txt) is a reference document from the template authors describing every theme variable (colors, fonts, sizes) and the exact CSS selector each one maps to in `main.css`. When asked to tweak colors/typography, consult it rather than grepping blindly — e.g. primary color `#1851f1` lives on a long selector list around the `/* PRIMARY COLOR */` block in `main.css`.

## Deployment

Pushing to `main` publishes to `https://shadowknight08.github.io/` via GitHub Pages. There is no staging environment — verify visually in a browser before pushing.
