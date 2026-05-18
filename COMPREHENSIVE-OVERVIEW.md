# Comprehensive Overview

**Start here if you are new to this repo.**

This document is a single narrative introduction to the Constellation Marketing website rebuild. It pulls together context from the rest of the documentation in `documents/` so you understand what you are working on, how the pieces fit, and where to look next, without reading eleven separate guides on day one.

---

## What this project is

**Constellation Marketing** is a law firm marketing agency (SEO, paid ads, web design, content, and related services for attorneys). The public website at [goconstellation.com](https://www.goconstellation.com) is the company’s main marketing and lead-generation surface. Most business value flows through **“book a call”** scheduling and supporting content (blog, city pages, case studies, podcasts) that ranks in search and builds trust.

The repository is named **`con-staging`**. It contains a **static rebuild** of that WordPress site using **[Astro 6](https://astro.build)**. There is roughly **one Astro file per URL** (~400 pages), no CMS, no database, and no user accounts. Content lives in Git as `.astro` files with HTML inside them. When you run a build, Astro emits plain HTML, CSS, and assets into `dist/`, and **Cloudflare Pages** serves those files from a CDN.

**Important context for timing:** Production traffic still runs on **WordPress + Divi** today. This Astro project deploys automatically to a **staging** Cloudflare Pages project (`con-staging`) on every push to `main`. Launching Astro to the live domain is a separate **DNS/hosting cutover**, not something that happens just because you merged code. Until that cutover, treat this repo as the future site and WordPress as the current live site.

---

## The mental model: not a web app

If you are used to Next.js apps with APIs, auth, and a database, adjust expectations early.

| You might expect | What this repo actually is |
|------------------|------------------------------|
| A server handling requests | Pre-built HTML files on a CDN |
| A CMS or admin UI | Content edited in `.astro` files in Git |
| API routes | None (`output: 'static'` in `astro.config.mjs`) |
| Runtime JavaScript framework | Mostly static markup; minimal client JS |
| One design system | Three overlapping CSS layers from migration (see below) |

A visitor’s browser requests a path like `/book-call/`. Cloudflare returns the matching file from `dist/book-call/index.html`. No Node process runs per request. That simplicity is intentional: fast, cheap hosting and predictable behavior at scale.

Your day-to-day work usually falls into one of these buckets:

1. **Edit content**: text, images, or HTML in a page under `src/pages/`.
2. **Edit a template**: change how a *type* of page looks (blog shell, city layout, header).
3. **Edit styling**: Tailwind tokens, legacy Divi CSS, or inline styles carried over from WordPress.
4. **Run migration scripts**: optional Python/Node tools that scrape production or bulk-generate pages (less common once you are in maintenance mode).

When something “does not work,” ask: is it a **build error**, a **CSS specificity** issue, or a **third-party embed** (booking calendar, ClickUp form) that only behaves correctly on certain domains?

---

## Quick start

You need **Node.js 22.12+** and npm.

```bash
npm install
npm run dev    # http://localhost:4321
```

Open the homepage, click around the header nav, and open a blog post and a city page. That gives you a feel for the three main page families.

Before you open a PR that touches layout or global CSS, also run:

```bash
npm run build
npm run preview
```

CI only runs `npm run build`; if that fails, deploy fails. `preview` serves the same artifact Cloudflare will get.

Optional tools you may encounter later: **Python 3** (bulk page scripts), **BackstopJS** (visual regression), **Anthropic API key** (automated homepage diff-fix script only, not used at runtime).

---

## How the codebase is organized

```
con-staging/
├── src/
│   ├── pages/              # Routes (file path ≈ URL)
│   ├── layouts/
│   │   ├── BaseLayout.astro    # <html>, meta tags, global CSS links
│   │   └── templates/          # Page-type layouts (blog, city, homepage, …)
│   ├── components/         # SiteHeader, SiteFooter
│   └── styles/
│       ├── global.css          # Tailwind 4 + brand tokens + fonts
│       └── divi-theme-options.css
├── public/                 # Static assets copied to site root (/images, /css, /fonts)
├── scripts/                # Migration & QA helpers (not shipped to visitors)
├── functions/              # Cloudflare Pages middleware (staging basic auth)
├── documents/              # All project documentation
├── astro.config.mjs
└── .github/workflows/deploy.yml
```

**`src/pages/`** is the routing layer. `src/pages/index.astro` is `/`. `src/pages/blog/law-firm-seo.astro` is `/blog/law-firm-seo/`. Nested folders create nested URLs.

**`src/layouts/templates/`** holds full-page shells. A typical page file is thin: it imports a template and passes SEO props plus content.

**`public/`** is served as-is. Large migrated images live under `public/images/`. The legacy Divi stylesheet `public/css/production-extracted.css` (~488KB) loads on every page via `BaseLayout`.

**`scripts/`** is for migration and QA, not scrapers, Backstop helpers, an optional Claude-powered fix loop. Nothing in `scripts/` runs when a visitor loads the site.

---

## How a page is built

Almost every public page follows the same stack:

```
src/pages/your-page.astro
        ↓
Template (e.g. BlogPostTemplate, CityTemplate)
        ↓
BaseLayout  →  <head>, SEO meta, CSS
SiteHeader  →  navigation
[your content]
SiteFooter
```

**Example: homepage** (`src/pages/index.astro`):

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

**Example: blog post:** the page file sets `seoTitle`, `seoDescription`, `canonicalUrl`, and puts the article body as **HTML inside the template’s default slot**. There is no Markdown pipeline and no CMS API; what you see in the file is what ships.

### Templates you should know

There are **13 templates** in `src/layouts/templates/`. You rarely need all of them on day one. The common ones:

| Template | Used for |
|----------|----------|
| `HomepageTemplate` | Homepage (`index.astro`) |
| `BlogPostTemplate` | ~161 blog articles |
| `CityTemplate` | Geo landing pages (`attorney-marketing-*`, `law-firm-marketing-*`) |
| `PracticeAreaTemplate` | Long practice-area guides |
| `ServiceTemplate` | Service line pages (SEO, PPC, web design) |
| `CoreTemplate` | About, FAQ, privacy, blog index |
| `PodcastTemplate` | Podcast episodes with YouTube embed |
| `CaseStudyNarrativeTemplate` | Client stories |

Some high-value pages (`book-call`, `law-firm-seo-services`, `become-a-partner`) **do not use a template**; they are hand-built with `BaseLayout` + header/footer directly.

**Template examples:** routes under `/examples/` (e.g. `/examples/blog-post-example/`) show each template in isolation. Use them when learning props and layout.

Full prop reference: [TEMPLATES.md](./TEMPLATES.md).

### Shared chrome

- **`BaseLayout.astro`**: Document shell, title, meta description, canonical, Open Graph/Twitter tags, favicon, font preload, links to global and Divi CSS.
- **`SiteHeader.astro`**: Green pill nav: Results, Practice Areas, Services, Resources, Book a Call CTA.
- **`SiteFooter.astro`**: Footer links, social icons, sitemap link (still points at WordPress sitemap today).

Only two shared components exist besides layouts. Most “components” are still inline HTML inside templates from migration.

---

## The migration story (why things feel uneven)

This site was not greenfield. It was **scraped, exported, and rebuilt** from a WordPress + Divi production site while trying to preserve URLs, rankings, and visual parity.

Typical migration flow:

1. **Extract**: Scrape production HTML or export via [Refore](https://refore.ai) (homepage approach documented in [REFORE-APPROACH.md](../REFORE-APPROACH.md)).
2. **Transform**: Python/Node scripts generate `.astro` pages; images land in `public/images/` (often WebP).
3. **Verify**: BackstopJS screenshot comparison against production ([VISUAL-QA.md](./VISUAL-QA.md)).
4. **Iterate**: Manual fixes; optional `scripts/fix-loop.mjs` uses Claude to nudge homepage CSS.

That history explains several quirks you will hit immediately.

### Quirk 1: Two URLs for one page

During migration, the **file path in Astro** does not always match the **URL Google has indexed**.

Example: `src/pages/blog/law-firm-seo.astro` is served locally at `/blog/law-firm-seo/`, but its `canonicalUrl` may be `https://goconstellation.com/law-firm-seo/` because that is where WordPress always ranked.

Every indexable page should set an explicit **`canonicalUrl`** to the production URL you want to keep. If the Astro path differs from that URL, add a **redirect** in `astro.config.mjs`; do not change only the canonical.

When in doubt, grep for a similar page and copy its pattern. Details: [SEO.md](./SEO.md), [CONTENT-GUIDE.md](./CONTENT-GUIDE.md).

### Quirk 2: Three CSS layers

Styling is not one system. It is three layers loaded at once:

1. **Legacy Divi CSS**: `public/css/production-extracted.css` on every page (huge, from WordPress).
2. **Astro global CSS**: `src/styles/global.css` with Tailwind 4, brand colors, self-hosted Poppins; imports `divi-theme-options.css` (WordPress custom CSS snapshot).
3. **Template/page CSS**: Scoped `<style>` blocks and inline `style=""` on sections.

Newer pages (homepage v3, book-call) lean on inline styles and Poppins. Older migrated pages still depend on Divi class names. If a style change “does nothing,” you are probably fighting the wrong layer or specificity. Details: [STYLING.md](./STYLING.md).

### Quirk 3: Images still on WordPress

Many pages reference `https://goconstellation.com/wp-content/uploads/...` for photos and badges. That works while WordPress is up; it **breaks** if WordPress is turned off before those assets are copied to `public/images/`. High-traffic pages like `book-call.astro` and homepage variants are worth auditing first.

### Quirk 4: Dead WordPress forms in old HTML

Migrated HTML and CSS may still reference **Ninja Forms** (`admin-ajax.php`). Those forms do not work on Astro. Live lead capture should go through **`/book-call/`** (GoHighLevel) or **`/support-center/`** (ClickUp), not legacy form markup.

---

## SEO and metadata

SEO is **per-page frontmatter**, not a central CMS field.

| Field | Purpose |
|-------|---------|
| `seoTitle` | Browser tab title and social titles (often `Keyword \| Constellation Marketing`) |
| `seoDescription` | Meta description |
| `canonicalUrl` | Absolute production URL for `<link rel="canonical">` |

`BaseLayout` also sets Open Graph and Twitter Card tags. Default OG image: `/images/constellation-og.jpg`.

**Known bug:** `CoreTemplate` passes `seoTitle={...}` to `BaseLayout`, but `BaseLayout` expects `title={...}`. Pages like about-us, FAQ, privacy policy, and blog index may have broken `<title>` tags until that is fixed.

**Staging vs production:** `public/robots.txt` currently has `Disallow: /` (blocks all crawlers), appropriate for staging. Production launch must replace it and add a real sitemap. There is no `@astrojs/sitemap` in the project yet; the footer still links to WordPress’s `sitemap_index.xml`.

**Analytics:** Google Tag Manager (`GTM-PDGND8M`) runs on production WordPress but is **not** in `BaseLayout` yet. Launch requires porting GTM (or equivalent) into Astro.

Deep dive: [SEO.md](./SEO.md).

---

## Integrations (what talks to the outside world)

Most pages are static HTML with no third-party scripts. Integrations cluster on a few routes:

| Page | Service | How |
|------|---------|-----|
| `/book-call/` | **LeadConnector / GoHighLevel** | iframe + `msgsndr` embed script (primary conversion) |
| `/support-center/` | **ClickUp** | Hosted form iframe |
| `/podcasts/*` | **YouTube** | iframe embed; links to Spotify/Apple optional |
| Footer / thank-you | **Social** | Outbound links only (Facebook, Instagram, LinkedIn, X, YouTube) |

There is no sitewide chat widget, no HubSpot embed in templates, and no live Ninja Forms. Blog content may *mention* other products; that is editorial, not integration.

Dev-only: **Anthropic API** in `scripts/fix-loop.mjs` for visual QA automation; **BackstopJS** for screenshots. Neither runs for site visitors.

Details: [INTEGRATIONS.md](./INTEGRATIONS.md).

---

## Build, deploy, and environments

| Environment | What it is |
|-------------|------------|
| **Local** | `npm run dev` → `localhost:4321` |
| **Local production-like** | `npm run build` + `npm run preview` |
| **Staging** | Cloudflare Pages project `con-staging`, deploy on push to `main` |
| **Production (today)** | WordPress at `www.goconstellation.com` |
| **Production (future)** | Same Astro build, DNS pointed at Cloudflare |

**CI pipeline** (`.github/workflows/deploy.yml`):

```
push to main → npm ci → npm run build → wrangler pages deploy dist --project-name con-staging
```

Requires GitHub secret `CLOUDFLARE_API_TOKEN`. There is no `wrangler.toml`; the deploy command is inline in the workflow. PRs do not auto-deploy previews in the current setup.

**Staging protection:** `functions/_middleware.ts` enforces HTTP Basic Auth on the Cloudflare deployment when `BASIC_AUTH_USER` and `BASIC_AUTH_PASS` are set in Pages environment variables. Local dev does not use this middleware.

**Redirects** live in `astro.config.mjs` (a small set of long WordPress slugs → shorter Astro slugs today). Add more there when URLs change.

Details: [DEPLOYMENT.md](./DEPLOYMENT.md). Launch tasks: [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md).

---

## Content: where things live

There is no headless CMS. Content is versioned in Git.

| Content type | Location | How it gets there |
|--------------|----------|-------------------|
| Blog posts | `src/pages/blog/*.astro` | Manual edit or `scripts/build_blog_posts.py` |
| City pages | `src/pages/attorney-marketing-*.astro`, etc. | `scripts/build_city_pages.py` or manual |
| Homepage | `HomepageTemplate.astro` | Refore + manual work |
| Images | `public/images/` | Migration, often WebP |
| Podcasts / case studies | Per-page `.astro` files | Migration or manual |

Blog index (`blog/index.astro`) uses a **manual list** of posts, not every new blog file appears automatically.

Authoring conventions: [CONTENT-GUIDE.md](./CONTENT-GUIDE.md).

---

## Brand and typography (short reference)

Headings use **Poppins** (self-hosted woff2 in `public/fonts/`). Body text uses system UI fonts. Brand greens and navy are defined as Tailwind theme tokens in `global.css`, for example `#2a9f68` (green-dark) and `#1c2e4a` (navy used in heroes).

You will see these colors in inline styles across templates; they are the de facto design language even where Divi classes still exist.

---

## Things that will trip you up

1. **Assuming the Astro path is the public URL**: always check `canonicalUrl` and redirects.
2. **Editing only canonical without redirect**: splits SEO signals between two URLs.
3. **Trusting `npm run dev` alone before a big CSS change**: run `build` + `preview`.
4. **Breaking booking or support embeds**: test on the real Cloudflare hostname; widgets may restrict domains.
5. **Removing WordPress before migrating `wp-content` images**: widespread broken images.
6. **Expecting forms in migrated footers to work**: they pointed at WordPress.
7. **`CoreTemplate` title bug**: wrong `<title>` on several important pages until fixed.
8. **Import path depth**: `../` vs `../../` for templates depends on page folder depth ([TEMPLATES.md](./TEMPLATES.md#import-paths-easy-to-get-wrong)).

---

## Suggested first week

**Day 1: Orient**

- Read this document.
- Run `npm install`, `npm run dev`, click through header nav.
- Open `astro.config.mjs`, `BaseLayout.astro`, `SiteHeader.astro`, one blog post, one city page.

**Day 2: Edit something small**

- Fix a typo or meta description on one page; run `npm run build`.
- View source on that page locally; confirm `<title>`, canonical, and description.

**Day 3: Templates**

- Visit `/examples/` routes; read [TEMPLATES.md](./TEMPLATES.md) sections for `BlogPostTemplate` and `CityTemplate`.

**Day 4: Styling**

- Change something minor in `global.css`; understand why Divi CSS might override it ([STYLING.md](./STYLING.md)).

**Day 5: Pipeline**

- Trace `.github/workflows/deploy.yml`; open staging URL (with basic auth if configured).
- Test `/book-call/` and `/support-center/` embeds on staging.

**When you touch launch-related work**

- Use [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) as the single cutover runbook.

---

## Documentation map

Use this table to go deeper. You do not need every doc immediately.

| Document | Read when you need to… |
|----------|-------------------------|
| [ASTRO.md](./ASTRO.md) | Astro basics, static output, page/template stack |
| [DEVELOPMENT.md](./DEVELOPMENT.md) | Day-to-day commands, editing workflows, troubleshooting |
| [TEMPLATES.md](./TEMPLATES.md) | Props, slots, which template to pick |
| [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) | Add or edit blog posts, city pages, images |
| [STYLING.md](./STYLING.md) | CSS layers, brand tokens, Divi vs Tailwind |
| [SEO.md](./SEO.md) | Canonicals, redirects, robots, sitemaps |
| [INTEGRATIONS.md](./INTEGRATIONS.md) | GHL, ClickUp, YouTube, GTM gap |
| [DEPLOYMENT.md](./DEPLOYMENT.md) | CI/CD, Cloudflare, manual Wrangler deploy |
| [MIGRATION-TOOLING.md](./MIGRATION-TOOLING.md) | Python/Node scripts, Refore, Backstop |
| [VISUAL-QA.md](./VISUAL-QA.md) | Screenshot regression testing |
| [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) | Production cutover from WordPress |
| [SITE-OVERVIEW.md](./SITE-OVERVIEW.md) | Shorter architecture reference |
| [REFORE-APPROACH.md](../REFORE-APPROACH.md) | Homepage migration methodology |

---

## Key files to bookmark

1. `astro.config.mjs`: static output, site URL, redirects  
2. `src/layouts/BaseLayout.astro`: global `<head>` and CSS  
3. `src/components/SiteHeader.astro`: site navigation / IA  
4. `src/pages/book-call.astro`: primary conversion + GHL embed  
5. `src/layouts/templates/HomepageTemplate.astro`: largest template  
6. `src/layouts/templates/CityTemplate.astro`: geo page pattern (~138 city pages)  
7. `src/pages/blog/law-firm-seo.astro`: typical blog + canonical mismatch example  
8. `.github/workflows/deploy.yml`: how code becomes staging  

---

## Summary

You are maintaining a **large, static, SEO-driven marketing site** mid-migration from WordPress. Pages are Astro files, not CMS entries. Templates provide structure; HTML content often still reflects Divi-era markup. Staging deploys automatically from `main`; production cutover is a planned DNS and configuration event, not an automatic consequence of merging code.

The highest-impact routes are **`/book-call/`** (revenue), **homepage and service pages** (positioning), and **blog/city content** (organic traffic). When you change URLs, think in pairs: **file path + canonical + redirect**. When you change styles, think in layers: **Divi CSS vs global.css vs inline**. When you are unsure, grep the repo for a similar page and follow what it already does.

Welcome aboard; start with `npm run dev` and one small, safe edit.
