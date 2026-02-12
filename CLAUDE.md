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
- Service metadata for JSON-LD: `data/services.yml` (Spanish default), with language overrides in `data/{lang}/services.yml`
- Header/logo config in `data/globalheader.yml`
- Footer links configured in `config/_default/config.toml` under `[params.footer]`

### Service Pages
Four services in `content/services/`: `coaching-ex`, `coaching-per`, `maas`, `consultancy`. Each has HTML content files per language (`main.html`, `main.en.html`, etc.).

### Blog Posts
- Posts are in `content/posts/` with language suffixes: `post-slug.md` (Spanish default), `post-slug.en.md`, `post-slug.fr.md`, `post-slug.ca.md`
- Front matter includes: `title`, `image`, `summary`, `image_alt`, `date`, `tags`
- Post banner images go in `static/images/posts/{post-slug}/banner.png`
- Post list uses a 2-column CSS grid (`.post-grid`) that collapses to 1 column at 840px
- Post cards (`.post-card`) use the shared design system: translucent white bg, hover lift/shadow, image zoom on hover
- Tags render as green pill links (`.post-card-tag` / `.post-tag`) matching `showcase-pill` style
- Card template: `layouts/_default/shortblock.html` overrides the theme's float-based layout
- Summaries are clamped to 3 lines via `-webkit-line-clamp` for consistent card heights

### External Integrations
- Form backend: un-static.com
- reCAPTCHA v2 for form validation
- Newsletter: Brevo (Sendinblue)
- Analytics: Google Analytics, Umami Cloud, Ahrefs

### Content Security Policy
- CSP is configured via meta tag in `layouts/partials/head/custom.html`
- When adding new external scripts or services, update the CSP directives accordingly (`script-src`, `connect-src`, `frame-src`, etc.)

### CSS Design System
The service pages share a consistent set of reusable UI components defined across SCSS partials:

**Color palette:**
- `#1e3f2b` — dark green (primary text, backgrounds)
- `#d7f7e4` — light green (pill gradients, accents)
- `#55f997` — accent green (borders, highlights)
- `rgba(255, 255, 255, 0.6)` — translucent white (card backgrounds, with `backdrop-filter: blur`)

**Homepage highlight cards** (in `assets/sass/banner.scss` + `assets/sass/custom.scss`):
- "Credo cards" design: glassmorphism bg, green top accent bar (`::before`), icon in green circle backdrop
- Icons at 72px inside a 120px soft green circle (`linear-gradient(135deg, rgba(215,247,228,0.5), rgba(85,249,151,0.15))`)
- Center-aligned text (not justified) for short philosophical statements
- Equal-height cards via flexbox (`display: flex; flex-direction: column; height: 100%`)
- No ripple effect — clean hover with lift + shadow + icon scale only
- Section background uses gradient `linear-gradient(180deg, #e9e9e7, #eef6f0)` bridging banner gray to invitation mint

**Shared components** (defined in `assets/sass/maas.scss`, used across all service pages):
- `showcase-grid` / `showcase-card` / `showcase-pills` — feature cards with icon, title, description, and pill tags
- `showcase-grid-3` — 3-column variant for coaching-per
- `minimal-pricing` / `minimal-card` — cards with green left border accent, used for pricing and benefits
- `minimal-card-options` / `minimal-card-option` — sub-options within minimal cards
- `compare-table` / `compare-col` — comparison grid (4-column default, `compare-table-3` for 3-column)
- `benefits-list` — checklist with `circle-icon` checkmarks inside cards

**Consultancy-specific components** (in `assets/sass/consultancy.scss`):
- `objectives-grid` / `objective-card` — 5-column grid with dark green top border accent
- `timeline` / `timeline-item` — vertical timeline with numbered circles
- `deliverables-grid` / `deliverable-card` — 5-column grid with icons, titles, and descriptions

**Post components** (in `assets/sass/posts.scss`):
- `post-grid` — responsive 2-column grid (1-column at 840px)
- `post-card` / `post-card-image` / `post-card-body` — top-image cards with hover lift, image zoom
- `post-card-tag` / `post-tag` — green gradient pill links for tags
- `post-featured-image` — single post featured image with border-radius

**Contact components** (in `assets/sass/contact.scss`):
- `contact-form-section` — translucent white form container with green top accent border
- `form-input` / `form-textarea` / `form-select` — styled inputs with green focus ring
- `form-button` — submit button matching site-wide button style
- `cta-section` — dark green CTA with animated radial gradient background
- `submit-contact-grid` — flex layout centering button and reCAPTCHA

**Utilities** (in `assets/sass/markers.scss`):
- `marker-highlight` — green background text highlight
- `underline-highlight` — green underline accent
- `underline-highlight-dark` — dark green underline; animates (width 0→100%) when inside `.animate-on-scroll.is-visible` containers
- `side-border-highlight` — left border with green gradient background
- `circle-icon` — green gradient circle with checkmark

### Service Page Icons
- Style: filled/solid dark green (`#1e3f2b`) on transparent background
- Size: 512x512 PNG
- Located in `static/images/services/{service-name}/`
- When replacing icons from AI-generated composites, use alpha-channel thresholding to split cleanly:
  ```bash
  magick composite.png -crop {w}x{h}+{x}+{y} +repage \
    -channel A -threshold 50% +channel \
    -trim +repage -resize 440x440 \
    -gravity center -background none -extent 512x512 output.png
  ```

### CSS and Per-Language Content
CSS is compiled once for all languages (single `style.css`). For per-language dynamic content, use Hugo templates to inject values from data files.

### Font Loading
- Fonts (Source Sans Pro, FontAwesome) are defined in the theme but overridden in `static/css/` to set `font-display: swap`
- This ensures text displays immediately with fallback fonts, avoiding invisible text flash (FOIT)
- Override files: `static/css/font-source-sans-pro.css`, `static/css/fontawesome-all.min.css`
- Critical fonts are preloaded in `layouts/partials/head/custom.html`
- CSS also loaded via `<link>` tags in `head.html` to enable parallel loading (the theme's @import will use cached files)
- If the theme updates its font files, these overrides may need to be regenerated

### Smooth Scrolling
- `html { scroll-behavior: smooth }` in `assets/sass/custom.scss` enables smooth navigation for all internal links

### JavaScript Loading
- JS scripts are loaded with `defer` attribute via `layouts/partials/scripts.html` override
- This prevents render-blocking while maintaining execution order

### Header Logo Styling
- The header uses `<h1>` on homepage and `<p>` on other pages (for SEO H1 structure)
- These elements have different browser defaults, so `assets/sass/logo.scss` explicitly sets identical styles (font-size, line-height, display, padding) on both to ensure consistent header height across all pages
- Padding uses `em` units, so font-size must also be matched

### Image Optimization
Images use WebP format with PNG fallback for browser compatibility. WebP provides 25-80% smaller files than PNG.

**Adding new images:**
```bash
# Resize to 2x display size (for retina) and convert to WebP
magick input.png -resize 224x224 -quality 85 output.webp

# Example: icon displayed at 112x112 → resize to 224x224
magick myicon.png -resize 224x224 -quality 85 myicon.webp
```

**Size guidelines (2x for retina):**
- Icons (displayed 112x112): resize to 224x224
- Background images: resize to 2x expected display size
- Always keep PNG as fallback, reference WebP in `<picture>` element

**Template usage:**
```html
<picture>
  <source srcset="/images/example.webp" type="image/webp">
  <img src="/images/example.png" alt="Description">
</picture>
```

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
- **Meta descriptions**: Fallback chain in `layouts/partials/head/custom.html`:
  1. `.Description` (front matter)
  2. `.Params.summary` (blog posts)
  3. `.Summary` (Hugo auto-generated, truncated to 160 chars)
  4. `.Site.Params.description` (site-wide fallback from `config.*.toml`)
- **H1 structure**: Each page has exactly one H1:
  - Homepage: Logo/site title (in header)
  - Other pages: Page title (in content area)
  - Configured in `layouts/partials/header.html` (conditional h1/p for logo) and `layouts/_default/single.html`, `layouts/_default/list.html` (h1 for page titles)
- **Hreflang tags**: Implemented in `layouts/partials/head/custom.html`
- **x-default hreflang**: Points to Spanish (hardcoded as "es" in template)
- **Canonical tags**: Explicit `<link rel="canonical">` on all pages in `layouts/partials/head/custom.html`
- **Page titles**: Conditional brand suffix " | Marta Arpa" added only when title ≤ 45 chars (keeps total under Google's ~60 char limit). Configured in `layouts/partials/head/head.html`
- **JSON-LD Structured Data**: Schema.org markup in `layouts/partials/schema/`:
  - `main.html` - Controller that includes appropriate schemas per page type
  - `organization.html` - Organization schema (homepage only)
  - `service.html` - Service schema (service pages, uses `data/services.yml`)
  - `blogposting.html` - BlogPosting schema (blog posts, includes author and publisher with logo)
  - `breadcrumb.html` - BreadcrumbList schema (all non-home pages)

## Theme

Uses the Arcana theme as a git submodule. **Do not modify files in `themes/arcana/`** - create overrides in `layouts/` instead.

Custom templates in `layouts/` override theme defaults:
- `layouts/_default/single.html` - Single page template with H1, tag pills, featured image with border-radius
- `layouts/_default/list.html` - List page template with H1, post-grid wrapper
- `layouts/_default/shortblock.html` - Post card with top image, summary, tag pills (replaces theme's float layout)
- `layouts/partials/homepage/blocks/highlights.html` - Credo cards without ripple effect, icon in `<picture>` with WebP
- `layouts/partials/header.html` - Header with conditional H1/P for logo
- `layouts/partials/head/head.html` - Page title with conditional brand suffix
- `layouts/partials/head/custom.html` - CSP, analytics, meta tags, meta descriptions, hreflang tags, canonical tags, JSON-LD schemas
- `layouts/partials/schema/*.html` - JSON-LD structured data partials (organization, service, blogposting, breadcrumb)

Initialize submodules after cloning:

```bash
git submodule update --init --recursive
```

## Git Workflow

**Never push directly to `main`.** All changes must go through a feature branch and pull request:

1. Create a feature branch from `main`
2. Make changes and commit to the branch
3. Open a pull request for review
4. Merge to `main` after approval

## Deployment

Automatic deployment to GitHub Pages via `.github/workflows/hugo.yaml` on push to `main` branch.
