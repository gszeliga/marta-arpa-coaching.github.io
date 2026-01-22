# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hugo-based static website for Marta Arpa's coaching and consulting business. Multilingual site (Spanish default, English, French, Catalan) deployed to GitHub Pages at martaarpa.com.

## Development Commands

```bash
# Local development server with hot reload
hugo server

# Build production site (outputs to /public)
hugo --environment production --gc --minify

# Linting (Trunk.io)
trunk check          # Run all linters
trunk fmt            # Auto-format code
```

**Required versions**: Hugo Extended v0.154.2, Dart Sass v1.89.2

## Architecture

### Multilingual Content Structure
- Content files use language suffixes: `_index.es.md`, `_index.en.md`, `_index.fr.md`, `_index.ca.md`
- Language configs in `config/_default/config.*.toml`
- Translation strings in `i18n/*.toml`
- Spanish is the default language (`defaultContentLanguage = "es"`)
- Catalan uses language key `ca` (RFC 5646 code), matching the i18n file `ca.toml`

### Directory Layout
- `content/` - Markdown and HTML source content
- `layouts/` - Hugo templates extending the Arcana theme
- `assets/sass/` - Custom SCSS stylesheets
- `data/` - YAML data files (homepage blocks, header config)
- `themes/arcana/` - Base theme (git submodule)
- `static/` - Static assets (images, fonts)

### Data-Driven Components
- Homepage blocks: `data/homepage.yml` (Spanish default), with language overrides in `data/{lang}/homepage.yml` (en, fr, ca)
- Header/logo config in `data/globalheader.yml`
- Footer links configured in `config/_default/config.toml` under `[params.footer]`

### Service Pages
Four services in `content/services/`: `coaching-ex`, `coaching-per`, `maas`, `consultancy`. Each has HTML content files per language (`main.html`, `main.en.html`, etc.).

### Blog Posts
- Posts are in `content/posts/` with language suffixes: `post-slug.md` (Spanish default), `post-slug.en.md`, `post-slug.fr.md`, `post-slug.ca.md`
- Front matter includes: `title`, `image`, `summary`, `image_alt`, `date`, `tags`
- Post banner images go in `static/images/posts/{post-slug}/banner.png`

### External Integrations
- Form backend: un-static.com
- reCAPTCHA v2 for form validation
- Newsletter: Brevo (Sendinblue)
- Analytics: Google Analytics + Umami Cloud

### CSS and Per-Language Content
CSS is compiled once for all languages (single `style.css`). For per-language dynamic content, use Hugo templates to inject values from data files.

### Banner Images
- Uses `<picture>` element with responsive sources for SEO/accessibility
- Each `homepage.yml` has `image`, `image_crop`, and `image_alt` fields for localized images and alt text
- At 768px and below, the cropped version (`image_crop`) is displayed; above 768px, the full image (`image`) is used
- Responsive heights and widths are configured in `assets/sass/banner.scss`

### URL Conventions
- All internal URLs must include trailing slashes (e.g., `/services/coaching-ex/main/` not `/services/coaching-ex/main`)
- This applies to: menu configs (`menus.*.toml`), homepage data files, footer links, and any hardcoded URLs
- Missing trailing slashes cause 301 redirects which impact SEO

### SEO Configuration
- Meta descriptions: `[params].description` in each `config/_default/config.*.toml` (keep between 110-160 chars)
- Hreflang tags: Implemented in `layouts/partials/head/custom.html`
- x-default hreflang points to Spanish (hardcoded as "es" in template)
- Page titles: Conditional brand suffix " | Marta Arpa" added only when title ≤ 45 chars (keeps total under Google's ~60 char limit). Configured in `layouts/partials/head/head.html`

## Theme

Uses the Arcana theme as a git submodule. Custom templates in `layouts/` override theme defaults:
- `layouts/partials/head/head.html` - Page title with conditional brand suffix
- `layouts/partials/head/custom.html` - Analytics, meta tags, hreflang tags

Initialize submodules after cloning:

```bash
git submodule update --init --recursive
```

## Deployment

Automatic deployment to GitHub Pages via `.github/workflows/hugo.yaml` on push to `main` branch.
