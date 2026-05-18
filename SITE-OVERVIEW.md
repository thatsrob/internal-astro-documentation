# Constellation Marketing: Site Overview

**Repository:** `con-staging`  
**Production site:** [goconstellation.com](https://www.goconstellation.com)  
**Staging deploy:** Cloudflare Pages project `con-staging`

This document describes what the site is, how it is built, and how content flows from development to production.

**For day-to-day setup, commands, and editing workflows, see [DEVELOPMENT.md](./DEVELOPMENT.md).**

---

## What this site is

Constellation Marketing is a **law firm marketing agency** (SEO, PPC, web design, content, etc.). The website is a **marketing and SEO platform**: it attracts attorneys, explains services, and drives conversions, primarily through **“Book a call”** flows.

This repo is a **static rebuild** of the production WordPress/Divi site using **Astro**. It is not a web application with users, APIs, or a database. Every page is pre-rendered HTML served from a CDN.

---

## Tech stack

| Layer | Technology |
|-------|------------|
| Framework | [Astro 6](https://astro.build) (`output: 'static'`) |
| Styling | Tailwind CSS 4 + legacy Divi CSS (`public/css/production-extracted.css`) |
| Fonts | Self-hosted Poppins (headings), system UI (body) |
| Runtime | Node.js ≥ 22.12 |
| Hosting | Cloudflare Pages |
| CI/CD | GitHub Actions (build on push to `main`, deploy via Wrangler) |
| Visual QA | BackstopJS (screenshot diff vs production) |
| Migration tooling | Python scrapers, Refore exports, optional Claude-assisted fix loop |

**Not included:** React/Vue, SSR, API routes, CMS, database, or server-side auth.

---

## Architecture at a glance

```
WordPress/Divi (production)
        │
        ▼  scrape / Refore export / scripts
src/pages/*.astro  +  templates  +  public/assets
        │
        ▼  npm run build
dist/  (static HTML, CSS, images)
        │
        ▼  GitHub Actions → Cloudflare Pages
CDN serves files to visitors
```

A visitor request never hits application logic; only static files from Cloudflare.

---

## Project structure

```
/
├── src/
│   ├── pages/           # One .astro file per URL (file-based routing)
│   ├── layouts/
│   │   ├── BaseLayout.astro
│   │   └── templates/   # Page-type layouts (homepage, blog, city, etc.)
│   ├── components/      # SiteHeader, SiteFooter
│   └── styles/
│       └── global.css   # Tailwind + brand tokens
├── public/
│   ├── images/          # Migrated assets (often WebP)
│   ├── css/             # production-extracted.css (Divi legacy styles)
│   └── fonts/
├── scripts/             # Migration & QA helpers (not runtime)
├── backstop_data/       # Visual regression artifacts
├── astro.config.mjs
└── .github/workflows/deploy.yml
```

---

## Routing

Astro maps files under `src/pages/` to URLs:

| File | URL |
|------|-----|
| `src/pages/index.astro` | `/` |
| `src/pages/blog/law-firm-seo.astro` | `/blog/law-firm-seo/` |
| `src/pages/attorney-marketing-austin.astro` | `/attorney-marketing-austin/` |

**Scale (approximate):**

- ~396 total pages
- ~161 blog posts
- ~66 `attorney-marketing-*` city pages
- ~72 `law-firm-marketing-*` city pages
- Plus practice areas, case studies, podcasts, nested guides, legal pages, etc.

URL redirects for renamed paths are defined in `astro.config.mjs`.

---

## How pages are built

Most pages follow the same pattern:

1. A **thin page file** in `src/pages/` sets SEO props (title, description, canonical URL).
2. It imports a **template** from `src/layouts/templates/`.
3. The template wraps content in `BaseLayout`, `SiteHeader`, and `SiteFooter`.

**Example: homepage:**

```astro
---
import HomepageTemplate from '../layouts/templates/HomepageTemplate.astro';
---
<HomepageTemplate
  seoTitle="..."
  seoDescription="..."
  canonicalUrl="https://goconstellation.com/"
/>
```

**Example: blog post:** props + chapter list + **HTML body** inside the template slot (content lives in the `.astro` file, not Markdown or a CMS).

---

## Page types and templates

Full inventory with layout patterns, standalone pages, and sheet vs repo counts: **[PAGE-INVENTORY.md](./PAGE-INVENTORY.md)**. Post-migration work: **[TODO.md](./TODO.md)**.

| Template | Purpose | Scale (approx.) |
|----------|---------|-----------------|
| `HomepageTemplate` (+ v2, v3) | Homepage and migration variants | 1 live |
| `CityTemplate` | Geo landing pages (e.g. Atlanta, Austin) | ~118–121 |
| `BlogPostTemplate` | Blog articles | ~132 (sheet) / ~160 (repo) |
| `PracticeAreaTemplate` | Long-form practice-area guides | ~9–12 |
| `CoreTemplate` | About, FAQ, blog/podcast index, legal | ~6+ |
| `ServiceTemplate` | Service line pages (SEO, PPC, branding, etc.) | ~6 built / ~18 manifest |
| `CaseStudyNarrativeTemplate` | Client success stories | ~27–29 |
| `PodcastTemplate` | Podcast episodes | ~22–23 |
| `BookACallTemplate` | Consultation landing (template); live `/book-call/` is custom | 1 |
| `GuidesTemplate` / `AgencyReviewTemplate` | Pillar guides and agency reviews | Few |

**Standalone pages** (not bulk-generated): `index`, `about-us`, `blog/index`, `podcasts/index`, `case-studies`, `services`, `book-call`, `faq`, plus three custom flagship service pages (`law-firm-seo-services`, `law-firm-ppc`, `law-firm-web-design`). See [PAGE-INVENTORY.md](./PAGE-INVENTORY.md#standalone-pages).

Shared layout:

- **`BaseLayout`**: HTML shell, meta tags, Open Graph, canonical, global CSS link
- **`SiteHeader`**: Main nav (Results, Practice Areas, Services, Resources)
- **`SiteFooter`**: Footer links and branding

---

## Styling approach

The site is in a **migration phase** with three styling layers (legacy Divi CSS, Tailwind/global CSS, and template inline styles). See [STYLING.md](./STYLING.md) for the full guide, brand palette, and debugging tips.

---

## Content sources

| Content type | Where it lives | How it gets there |
|--------------|----------------|-------------------|
| Blog posts | HTML inside `src/pages/blog/*.astro` | Manual edit or `scripts/build_blog_posts.py` (scrapes production) |
| City pages | Props + `CityTemplate` | `scripts/build_city_pages.py` |
| Homepage | `HomepageTemplate.astro` | Refore export + manual adaptation ([REFORE-APPROACH.md](../REFORE-APPROACH.md)) |
| Images | `public/images/` | Migrated from WordPress; often converted to WebP |
| Podcasts / case studies | Per-page `.astro` files | Migration scripts or manual |

There is **no headless CMS**; content is versioned in Git.

---

## Build and deploy

**Local development:**

```bash
npm install
npm run dev      # http://localhost:4321
npm run build    # output → dist/
npm run preview  # serve dist/ locally
```

**Production deploy:** On push to `main`, GitHub Actions runs `npm run build` and deploys `dist/` to Cloudflare Pages (`con-staging`). See [DEPLOYMENT.md](./DEPLOYMENT.md) for CI secrets, manual Wrangler deploy, and production cutover.

Site URL for canonicals/sitemap is set in `astro.config.mjs` as `https://www.goconstellation.com`.

---

## Migration and QA workflow

The site is being moved from **WordPress + Divi** to **Astro** while preserving SEO URLs and visual design.

1. **Extract**: Scrape production or use [Refore](https://refore.ai) to export HTML/CSS.
2. **Transform**: Generate `.astro` pages and optimize images.
3. **Verify**: BackstopJS compares staging sections to production ([VISUAL-QA.md](./VISUAL-QA.md)).
4. **Iterate**: Optional `scripts/fix-loop.mjs` uses Claude API to reduce visual diffs on the homepage.

Key docs and tools:

- [MIGRATION-TOOLING.md](./MIGRATION-TOOLING.md): All migration scripts, Backstop, and workflows
- [REFORE-APPROACH.md](../REFORE-APPROACH.md): Homepage migration methodology
- `backstop.config.cjs`: Visual regression config
- `scripts/split-sections.mjs`: Splits Divi HTML into sections
- `scripts/build_city_pages.py` / `build_blog_posts.py`: Bulk page generation

---

## What this codebase does *not* include

- User accounts or admin UI
- API routes or server-side logic
- Database or CMS integration
- Runtime AI (Anthropic SDK is dev-tooling only)

---

## Documentation index

| Document | Purpose |
|----------|---------|
| [TODO.md](./TODO.md) | **Post-migration**: pages to edit, QA, launch phases |
| [PAGE-INVENTORY.md](./PAGE-INVENTORY.md) | Migration tracker templates, standalone pages, counts |
| [COMPREHENSIVE-OVERVIEW.md](./COMPREHENSIVE-OVERVIEW.md) | **Start here**: narrative onboarding for new developers |
| [ASTRO.md](./ASTRO.md) | Astro concepts and how they apply on this site |
| [SITE-OVERVIEW.md](./SITE-OVERVIEW.md) | Architecture and stack (this file) |
| [DEVELOPMENT.md](./DEVELOPMENT.md) | Local setup, commands, editing, troubleshooting |
| [TEMPLATES.md](./TEMPLATES.md) | Page templates, props, slots, which to use when |
| [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) | Adding and editing blog posts, city pages, images |
| [SEO.md](./SEO.md) | Titles, canonicals, redirects, sitemaps, robots, cutover checklist |
| [INTEGRATIONS.md](./INTEGRATIONS.md) | GHL booking, ClickUp, YouTube, GTM gap, Cloudflare auth |
| [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) | WordPress → Astro production cutover (master checklist) |
| [MIGRATION-TOOLING.md](./MIGRATION-TOOLING.md) | Scripts, Refore, Backstop, bulk migration workflows |
| [STYLING.md](./STYLING.md) | CSS layers, brand tokens, Divi vs Tailwind, debugging |
| [DEPLOYMENT.md](./DEPLOYMENT.md) | Cloudflare Pages, GitHub Actions, cutover |
| [VISUAL-QA.md](./VISUAL-QA.md) | BackstopJS homepage screenshots, fix-loop |

## Key files to read first

1. `astro.config.mjs`: Static output, site URL, redirects
2. `src/layouts/BaseLayout.astro`: Global HTML shell
3. `src/components/SiteHeader.astro`: Site navigation / IA
4. `src/layouts/templates/HomepageTemplate.astro`: Most complex page
5. `src/layouts/templates/CityTemplate.astro`: Geo page pattern
6. `src/pages/blog/law-firm-seo.astro`: Typical blog structure
7. [REFORE-APPROACH.md](../REFORE-APPROACH.md): Migration approach
8. `.github/workflows/deploy.yml`: Deploy pipeline
