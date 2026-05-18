# Styling Guide

This guide explains **how CSS works in this project**: what loads on every page, where brand styles live, and how to change layout without fighting the WordPress migration legacy.

For templates and page structure, see [TEMPLATES.md](./TEMPLATES.md). For Refore/Divi migration context, see [REFORE-APPROACH.md](../REFORE-APPROACH.md) and [MIGRATION-TOOLING.md](./MIGRATION-TOOLING.md).

---

## The big picture: three layers at once

This site is mid-migration from **WordPress + Divi**. Styling is intentionally layered, not a single design system.

```
Every page
    │
    ├─ 1. Legacy Divi CSS     public/css/production-extracted.css  (~488KB, all pages)
    │
    ├─ 2. Astro global CSS    src/styles/global.css + Tailwind 4
    │       └─ imports divi-theme-options.css (WP custom CSS snapshot)
    │
    └─ 3. Page/template CSS   Scoped <style> blocks, inline style="", component CSS
```

**Why it exists:** Divi classes and production rules keep migrated HTML looking correct. Tailwind and `global.css` handle newer, shared tokens. Templates add layout-specific rules that were copied from production.

**Your job when styling:** Know which layer owns the element you are changing. Editing the wrong layer causes “nothing changed” or unexpected side effects elsewhere.

---

## What loads on every page

`BaseLayout.astro` is the entry point:

```astro
import '../styles/global.css';
...
<link rel="stylesheet" href="/css/production-extracted.css" />
<slot name="head" />
```

| Asset | Source | Role |
|-------|--------|------|
| `global.css` | Bundled by Astro/Vite | Tailwind, fonts, base typography, theme tokens |
| `divi-theme-options.css` | Imported inside `global.css` | Sitewide custom CSS from WP Divi Theme Options |
| `production-extracted.css` | Static file in `public/css/` | Full Divi framework snapshot for migrated markup |
| Per-page `<style slot="head">` | Templates / pages | Overrides (e.g. `body { background: #efefef }`) |

**Load order:** Astro-injected CSS (global + scoped) competes with `production-extracted.css` depending on specificity. Inline `style=""` usually wins for one-off layout.

### Page-specific Divi CSS files (not auto-loaded)

`public/css/` also contains large per-template exports:

| File | ~Size | Typical source page |
|------|-------|-------------------|
| `divi-homepage.css` | 452KB | Homepage |
| `divi-city.css` | 416KB | City pages |
| `divi-blog-post.css` | 411KB | Blog |
| `divi-practice-area.css` | 424KB | Practice areas |
| `divi-service.css` | 455KB | Services |

These are **reference / fallback** artifacts from migration. They are **not** linked in `BaseLayout` today; only `production-extracted.css` is global. City/blog/homepage styling is duplicated into template `<style>` blocks or covered by the combined extract.

To use a page-specific file temporarily, add via `<link>` in that template’s `slot="head"`; then consider merging useful rules into the template or trimming the file.

---

## Brand colors and typography

### Official palette (Tailwind `@theme`)

Defined in `src/styles/global.css`:

| Token | Hex | Typical use |
|-------|-----|-------------|
| `--color-green` | `#4fbc85` | Lighter green accents |
| `--color-green-dark` | `#2a9f68` | Primary brand green (header pill, CTAs, links) |
| `--color-green-light` | `#e8f7f0` | Soft green backgrounds |
| `--color-gray-light` | `#f3f4f4` | Page section backgrounds |
| `--color-navy` | `#1a2332` | Dark text / UI (theme token) |
| `--color-text` | `#666666` | Body copy |
| `--color-text-dark` | `#222222` | Headings in global base |

Templates also use **`#1c2e4a`** heavily (dark blue hero backgrounds, article headings). Treat it as the **marketing navy** alongside `--color-navy`.

### Hardcoded vs tokens

Most migrated templates use **hex literals in inline styles** (`#2a9f68`, `#1c2e4a`, `#f3f4f4`) rather than `var(--color-green-dark)`. That matches production copy-paste.

**For new UI:** Prefer Tailwind utilities where Tailwind is already in use, or CSS variables from `@theme` in `global.css` for consistency.

```html
<!-- Tailwind 4 (if using utilities in Astro) -->
<div class="bg-[--color-green-dark] text-white">

<!-- Or match existing templates -->
<div style="background: #2a9f68; color: #fff;">
```

### Fonts

| Role | Font stack | Files |
|------|------------|-------|
| **Headings** | Poppins 600/700/800 | `public/fonts/poppins-*.woff2` via `@font-face` in `global.css` |
| **Body** | System UI | `system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif` |

`BaseLayout` preloads `poppins-700.woff2` for LCP.

`@fontsource/poppins` is in `package.json` but **not imported** in code; self-hosted woff2 is what the site uses.

**Blog / practice articles** often set `font-family: 'Poppins'` on body text inside `.article-body` (not just headings).

---

## Tailwind CSS 4

Configured in `astro.config.mjs`:

```js
vite: {
  plugins: [tailwindcss()],
},
```

`src/styles/global.css` starts with:

```css
@import "tailwindcss";
```

**Current usage:** Light. Global base rules and `@theme` tokens matter more than utility classes across most templates. Utilities are available for new components (e.g. `class="site-container"` could be replaced with Tailwind spacing, but today `.site-container` is plain CSS in `global.css`).

**When to use Tailwind:**

- New small components or refactors where you want consistent spacing/colors
- Avoid mixing Tailwind and huge inline style blocks on the same element without a reason

**When not to rely on Tailwind alone:**

- Migrated Divi HTML with `et_pb_*` classes
- Homepage/city sections tuned pixel-by-pixel to production

---

## Astro styling mechanics

### Scoped vs global

| Pattern | Example | Effect |
|---------|---------|--------|
| `<style>` in `.astro` component | `SiteHeader.astro` | Astro scopes selectors (attributes added) |
| `<style is:global slot="head">` | `BlogPostTemplate` | Unscoped; injected in `<head>`; affects whole page |
| `<style slot="head">` | `PracticeAreaTemplate` | Unscoped body/hero overrides |
| Inline `style=""` | Homepage, many sections | Highest locality; copied from production |

Blog and practice templates use **`is:global`** so classes like `.article-body h2` apply to HTML in the default slot.

### `slot="head"`

Templates pass extra CSS to `BaseLayout`:

```astro
<style is:global slot="head">
  body { background: #efefef; }
  .article-body h2 { ... }
</style>
```

Use this for page-type-specific rules that should not live in the global bundle.

### `is:inline` scripts

Some templates (`CityTemplate`, carousels) use `<script is:inline>` for DOM behavior. That is separate from CSS but often sits next to large `<style>` blocks in the same file.

---

## Where styles live by area

### Site chrome

| Component | File | Notes |
|-----------|------|-------|
| Header | `src/components/SiteHeader.astro` | Scoped `.site-header`, `.pill-nav` (`#2a9f68` pill), dropdowns |
| Footer | `src/components/SiteFooter.astro` | Scoped footer styles |
| Global shell | `src/layouts/BaseLayout.astro` | No layout CSS; only links global + Divi extract |

Header is **sticky**, transparent wrapper, green pill nav at `max-width: 1340px`. Mobile menu toggles at `< 1024px`.

### Blog posts (`BlogPostTemplate`)

- Gray page background: `body { background: #efefef }`
- Article column: `.article-body` typography (h2–h4, lists, links, blockquotes)
- **Reusable content components** (use in HTML slot):

| Class | Purpose |
|-------|---------|
| `.stat-callout` | Dark navy box with green stat numbers |
| `.cta-inset` | Green rounded CTA block with white button |
| `.callout` | Light green insight box (see template for `.callout-label`) |

Example:

```html
<div class="stat-callout">
  <div class="stat-item">
    <div class="stat-num">85+</div>
    <div class="stat-label">Law firms served</div>
  </div>
</div>

<div class="cta-inset">
  <h3>Ready to grow your firm?</h3>
  <p>Short pitch.</p>
  <a href="/book-call/">Book a Call</a>
</div>
```

Migrated posts may still use old WordPress classes (`wp-image-*`, `aligncenter`); Divi extract styles those.

### Practice area guides (`PracticeAreaTemplate`)

Same **`.stat-callout`** and **`.cta-inset`** patterns as blog (duplicated in template CSS).

- Dark hero: `body { background: #1c2e4a }`
- Green reading progress bar (`#pa-progress-bar`) at top of viewport

### City pages (`CityTemplate`)

- Large scoped block: `.city .hero`, `.practice-grid`, `.faq-item`, etc. (~1000 lines in one `<style>`)
- Wrapper class **`.city`** on root; prefix keeps rules contained
- Responsive breakpoints at `900px` and `768px`
- Some content still loaded via inline script (case study cards from production URLs)

### Homepage (`HomepageTemplate`)

- Mostly **inline `style=""`** on sections
- Some named classes: `.constellation-faq`, etc.
- Section `data-section` attributes for Backstop alignment (not styling per se, but related to layout QA)

### Custom pages (`book-call`, `law-firm-seo-services`, …)

Hand-built inline styles + occasional scoped `<style>` blocks, not shared templates.

---

## `divi-theme-options.css`

Snapshot of **Divi → Theme Options → Custom CSS** (last noted 2026-03-30 in file header).

Includes:

- HubSpot form overrides (`#hsForm_...`)
- Legacy menu/CTA rules (`.menu-cta`, `.et_pb_menu_*`)
- Mobile overflow fixes
- Comments pointing at `.lfm-*` / `.constellation-*` used on city/FAQ blocks

Many selectors target **WordPress-only** DOM (`#main-content`, `.et_pb_section_0`). They may no-op on Astro pages, which is expected. Rules are kept for parity and in case migrated HTML still uses those classes.

**Edit when:** Sitewide behavior must match a WP custom CSS change. Prefer updating the Astro template if the selector only applies to one page type.

---

## `production-extracted.css`

Single bundled Divi stylesheet served to every page.

**Pros:**

- Migrated HTML with `et_pb_*`, `et_pb_row`, etc. often “just works”
- Avoids running 400KB+ through Vite for every page type separately

**Cons:**

- Large download (~488KB uncompressed)
- Global namespace; obscure Divi rules can affect unrelated markup
- Hard to know which rule “won” in DevTools without filtering

**When editing:** Prefer **not** editing this file for one-page fixes. Change the template or add a targeted override in `slot="head"`. Only regenerate/replace the extract when doing a deliberate sitewide Divi sync from production.

`REFORE-APPROACH.md` notes it was moved to `public/css/` to **bypass Astro build processing** on the raw Divi file.

---

## Decision guide: how should I style this?

| Situation | Approach |
|-----------|----------|
| New blog paragraph or list | Plain HTML; rely on `.article-body` rules |
| New blog callout / stats row | Use `.stat-callout` or `.cta-inset` |
| New practice area guide section | Same as blog; match `PracticeAreaTemplate` examples |
| New city page look | Edit `CityTemplate.astro` (affects all cities) or props/images only |
| Header/footer change | `SiteHeader.astro` / `SiteFooter.astro` scoped CSS |
| Sitewide font or body color | `src/styles/global.css` |
| Sitewide Divi/WP custom rule | `src/styles/divi-theme-options.css` |
| Match production homepage section exactly | Inline styles in `HomepageTemplate` or Refore export |
| Migrated HTML looks broken | Check if `et_pb_*` classes present; check Divi extract in DevTools |
| Brand-new marketing component (clean) | Tailwind + `@theme` tokens in `global.css` |

---

## Divi pitfalls (migration)

From production experience documented in [REFORE-APPROACH.md](../REFORE-APPROACH.md):

### `et_pb_equal_columns`

Divi CSS can apply `flex-direction: row-reverse`, reversing column order. **Fix:** Remove that class; use explicit `display: grid` or flex in your Astro markup.

### Class-dependent layout

If you strip Divi classes during cleanup (`build_astro_post.py` removes many), layout may collapse. Either keep classes or replace with equivalent CSS in the template.

### Specificity wars

`production-extracted.css` + inline styles + `!important` in Divi rules (e.g. theme options) can override your new rules. In DevTools, check which stylesheet wins before adding more `!important`.

### CORS / CDN fonts from Refore

Refore-hosted fonts may not load on staging; layout stays fine with system fallback. Self-host fonts in `public/fonts/` for production parity.

---

## Responsive design

Patterns vary by template:

| Template | Breakpoints (typical) |
|----------|------------------------|
| `SiteHeader` | Mobile menu `< 1024px` |
| `CityTemplate` | `900px`, `768px` |
| `BlogPostTemplate` | Article/chapter sidebar stacking in template CSS |
| Homepage | Per-section inline `clamp()` or media queries in `<style>` |

There is **no single global breakpoint system**; check the template you are editing.

---

## Debugging checklist

1. **Hard refresh** (Cmd+Shift+R); CSS is often cached.
2. **DevTools → Computed**: which file sets the property?
3. **Confirm layer**: Divi extract vs `global.css` vs template `head` slot vs inline.
4. **`npm run build`**: scoped Astro styles still apply in production. If the issue only appears in dev, check HMR.
5. **Compare production**: for migration pages, open goconstellation.com side-by-side.

---

## Making safe changes

### Do

- Match existing hex colors and Poppins weights when touching migrated sections
- Use template content classes (`.cta-inset`, `.stat-callout`) for long-form content
- Put one-page experiments in that page’s `slot="head"` style block
- Test mobile after header/city/homepage changes

### Avoid

- Editing `production-extracted.css` for a single typo on one blog post
- Removing Divi classes without replacing layout CSS
- Assuming Tailwind classes work inside scraped HTML that never imported utilities
- Adding huge new global rules to `divi-theme-options.css` for page-specific hacks

---

## File reference

| Path | Purpose |
|------|---------|
| `src/styles/global.css` | Tailwind import, fonts, `@theme`, base elements, `.site-container` |
| `src/styles/divi-theme-options.css` | WP Divi Theme Options custom CSS |
| `public/css/production-extracted.css` | Global Divi bundle (linked on every page) |
| `public/css/divi-*.css` | Per-page-type extracts (reference; not linked globally) |
| `public/fonts/poppins-*.woff2` | Self-hosted heading font |
| `src/layouts/BaseLayout.astro` | Stylesheet links |
| `src/components/SiteHeader.astro` | Nav styles |
| `src/layouts/templates/*.astro` | Page-type layout + typography |

---

## Related documentation

| Document | Topics |
|----------|--------|
| [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) | HTML in posts, image paths |
| [TEMPLATES.md](./TEMPLATES.md) | Which template owns which styles |
| [REFORE-APPROACH.md](../REFORE-APPROACH.md) | Refore CDN CSS, Divi HTML import |
| [DEVELOPMENT.md](./DEVELOPMENT.md) | Dev server, cache, build |
