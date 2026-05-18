# SEO Guide

This guide explains **how search-related metadata works** in the Astro site: titles, descriptions, canonicals, social tags, redirects, sitemaps, and migration pitfalls.

For content authoring conventions, see [CONTENT-GUIDE.md](./CONTENT-GUIDE.md). For deploy and cutover, see [DEPLOYMENT.md](./DEPLOYMENT.md) and [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md).

---

## How SEO metadata is generated

Almost all public pages flow through **`BaseLayout.astro`**, which outputs the document `<head>`:

| Output | Source |
|--------|--------|
| `<title>` | `title` prop |
| `<meta name="description">` | `description` prop (optional) |
| `<link rel="canonical">` | `canonicalUrl` prop |
| Open Graph tags | `title`, `description`, `ogImage`, `canonicalUrl` |
| Twitter Card tags | Same as OG |
| `lang` | Fixed `en-US` on `<html>` |

Templates pass SEO props from each page file, for example:

```astro
<BlogPostTemplate
  seoTitle="Winning Law Firm SEO: The Definitive Guide"
  seoDescription="Discover the ultimate guide to Winning Law Firm SEO..."
  canonicalUrl="https://goconstellation.com/law-firm-seo/"
/>
```

Inside `BlogPostTemplate`, that becomes `title={seoTitle}` on `BaseLayout`.

### Default canonical behavior

If you omit `canonicalUrl`, `BaseLayout` falls back to:

```js
canonicalUrl = Astro.url.href
```

That uses **whatever host you are viewing** (e.g. `http://localhost:4321/...` in dev). During migration, pages almost always set an explicit production `canonicalUrl` instead.

---

## The two-URL problem (migration)

| Concept | Controlled by | Example |
|---------|---------------|---------|
| **Astro file path** | `src/pages/...` | `src/pages/blog/law-firm-seo.astro` |
| **URL in the browser (staging)** | File path → route | `/blog/law-firm-seo/` |
| **Canonical (indexed URL)** | `canonicalUrl` prop | `https://goconstellation.com/law-firm-seo/` |

WordPress often ranked URLs **without** a `/blog/` prefix. The Astro file may live under `blog/` while the canonical points at the legacy production path.

**Rules:**

1. Decide the **production URL** that should rank (Search Console, sitemap, backlinks).
2. Set `canonicalUrl` to that exact URL (scheme, host, path, trailing slash).
3. If the Astro path differs, add a **redirect** in `astro.config.mjs` so visitors and bots reach one URL.

Changing only the canonical without a redirect splits signals between two URLs.

### Common patterns in this repo

| Pattern | Example |
|---------|---------|
| File under `/blog/`, canonical at root | `blog/law-firm-seo.astro` → canonical `/law-firm-seo/` |
| Nested guides | `law-firm-seo/benefit.astro` → canonical `/law-firm-seo/benefit/` |
| Podcast | `podcasts/evolve-or-die-....astro` → canonical may be `/evolve-or-die-.../` (root) |
| City pages | File `attorney-marketing-austin.astro` → canonical `/attorney-marketing-austin/` |

When adding a page, grep the codebase for a similar page and match its URL style.

---

## Field-by-field conventions

### `seoTitle` → `<title>`

- Shown in browser tab and search results.
- Common pattern: `Primary Keyword | Constellation Marketing`
- Shorter titles are fine for pillar content (e.g. `Winning Law Firm SEO: The Definitive Guide`).
- Aim for **~50–60 characters** visible in SERPs; Google may rewrite longer titles.

### `seoDescription`

- One or two sentences; include primary keyword naturally.
- Target **~150–160 characters**; not a hard limit.
- Omitting it removes the meta description tag entirely (`BaseLayout` only renders it when truthy).

### `canonicalUrl`

- **Absolute HTTPS URL** with trailing slash to match production:

```astro
canonicalUrl="https://goconstellation.com/attorney-marketing-austin/"
```

**Host consistency:** `astro.config.mjs` sets:

```js
site: 'https://www.goconstellation.com',
```

Many page canonicals use `https://goconstellation.com` **without** `www`. A few legal pages use `www`. Before cutover, pick one primary host in DNS/Search Console and align canonicals (and redirects) to match.

### `ogImage`

- Defaults to `/images/constellation-og.jpg` in `BaseLayout`.
- Per-page override: `ogImage="/images/your-hero.webp"`

**Gap:** OG/Twitter tags currently use the path as written. Social platforms prefer **absolute** URLs (`https://goconstellation.com/images/...`). If share previews break, resolve images to absolute URLs in `BaseLayout` or pass full URLs in props.

### Visible headings vs title

| Template | Visible H1 |
|----------|------------|
| `BlogPostTemplate` | `pageTitle` (defaults from `seoTitle` before `\|`) |
| `PracticeAreaTemplate` | `pageTitle` prop |
| `CityTemplate` | `Law Firm Marketing {city}` in hero |
| `CoreTemplate` | `heroTitle` prop |

Keep **one clear H1** per page. Migrated blog HTML sometimes includes extra `<h1>` or duplicate headings; remove duplicates in the body.

---

## Redirects

Configured in **`astro.config.mjs`** (build-time redirects for static hosting):

```js
redirects: {
  '/old-slug': '/new-slug',
  '/old-slug/': '/new-slug/',
},
```

Current entries rename long WordPress slugs to shorter Astro slugs (mass tort, employment law, workers comp).

**When to add redirects:**

- URL slug changes from WordPress
- Consolidating duplicate paths
- Enforcing trailing slash (add both with and without `/` if needed)

Redirects apply on the **Astro host** (staging after deploy, production after cutover), not on WordPress until traffic moves.

---

## Sitemaps

### Astro sitemap integration

**Not configured.** There is no `@astrojs/sitemap` in `package.json`. The build does not auto-generate `sitemap.xml` from routes.

### Footer link (WordPress)

`SiteFooter.astro` links to:

```html
<a href="/sitemap_index.xml">Sitemap</a>
```

That path served the **WordPress** sitemap index on production. On pure Astro staging it may **404** until you:

- Add an Astro-generated sitemap, or
- Proxy/serve a static sitemap at that path, or
- Update the footer link

### Cutover task: new sitemap

Before or at production launch:

1. Generate sitemap from all indexable routes (~400 pages).
2. Submit in Google Search Console.
3. Optionally add `sitemap.xml` reference in `robots.txt` (see below).

Options:

- Add `@astrojs/sitemap` to the Astro build, or
- Post-process `dist/` with a script that walks built HTML, or
- Maintain a generated XML in `public/` (gets stale without automation)

---

## `robots.txt`

**File:** `public/robots.txt`

```
User-agent: *
Disallow: /
```

This **blocks all crawlers** from the entire site. Appropriate for **staging** (`con-staging` on Cloudflare) so preview URLs are not indexed.

**Before production cutover**, replace with something like:

```
User-agent: *
Allow: /

Sitemap: https://www.goconstellation.com/sitemap.xml
```

(Use the same host you standardize on for `www` vs apex.)

---

## Structured data (Schema.org)

There is **no global** JSON-LD in `BaseLayout` (no sitewide `Organization` / `WebSite` block in Astro today).

Some **blog posts** include inline `<script type="application/ld+json">` in the HTML body (e.g. `LocalBusiness` examples in `law-firm-seo/local-seo-for-lawyers.astro`). That is content-level, not automatic.

**Cutover opportunities:**

- Sitewide `Organization` + `WebSite` in `BaseLayout` or a small SEO component
- `Article` schema on blog posts (author, date, image)
- `FAQPage` on FAQ sections where content supports it

Validate with [Google Rich Results Test](https://search.google.com/test/rich-results).

---

## Internal linking

- Prefer **root-relative** links for in-site navigation: `href="/book-call/"` so links work on staging and production.
- Migrated content often still has absolute `https://goconstellation.com/...` links; they work but are harder to test on staging hosts.
- **Blog index** (`blog/index.astro`) uses a **manual** post list, not every new post is listed automatically.
- **Header/footer** nav is defined in `SiteHeader.astro` / `SiteFooter.astro`.

---

## Pages to exclude from indexing

| Route | Recommendation |
|-------|----------------|
| `/examples/*` | Add `<meta name="robots" content="noindex">` via `slot="head"` before production, or remove routes |
| Thank-you / utility | Often `noindex` on production thank-you pages; confirm per page |
| Staging host | Keep `robots.txt` disallow until launch |

`examples/` routes are real build output today; do not rely on “hidden” URLs alone.

---

## Template-specific SEO notes

### `CoreTemplate` pages (`about-us`, `faq`, `privacy-policy`, `blog/index`)

`CoreTemplate` passes `seoTitle={seoTitle}` to `BaseLayout`, but **`BaseLayout` expects `title`**, not `seoTitle`. Until fixed, those pages may have **empty or wrong `<title>` tags** in the built HTML. Verify in view-source when editing these pages.

Correct pattern:

```astro
<BaseLayout title={seoTitle} description={seoDescription} canonicalUrl={canonicalUrl}>
```

### Custom pages (`book-call`, `law-firm-seo-services`, `case-studies`)

Some use `BaseLayout` directly with varying completeness. Audit view-source for title, description, and canonical when touching them.

### `book-call.astro`

May not pass full SEO props through `BaseLayout`; check before launch (high-intent landing page).

---

## `astro.config.mjs` `site` field

```js
site: 'https://www.goconstellation.com',
```

Used by Astro for absolute URL generation when features need it (e.g. future sitemap). It does **not** automatically fix canonicals or OG URLs in templates; those remain manual per page.

---

## Staging vs production SEO behavior

| Check | On staging Cloudflare | On production (after cutover) |
|-------|----------------------|-----------------------------|
| `robots.txt` | Likely blocks all | Must allow indexing |
| Canonical in HTML | Often `goconstellation.com` | Same (correct for continuity) |
| View in browser | Staging hostname | `www` or apex per DNS |
| Search Console | Separate property optional | Primary property |

Testing metadata locally: view-source on `localhost` after `npm run dev`; canonical will show production URL if set in the page file.

---

## Pre-launch SEO checklist

### Per new page

- [ ] `seoTitle` and `seoDescription` set
- [ ] `canonicalUrl` matches intended production URL exactly
- [ ] File path + redirect aligned with canonical if they differ
- [ ] One H1; logical heading order (`h2`, `h3`)
- [ ] Images have `alt` text
- [ ] Internal links use correct paths

### Sitewide (cutover)

- [ ] `robots.txt` allows crawling on production
- [ ] `www` vs non-`www` chosen; redirects in place
- [ ] Sitemap generated and submitted
- [ ] Footer sitemap link works or updated
- [ ] `CoreTemplate` / custom pages have valid `<title>`
- [ ] `/examples/` noindexed or removed
- [ ] OG images use absolute URLs if share previews matter
- [ ] Redirect map complete in `astro.config.mjs` + Cloudflare rules for edge cases
- [ ] Search Console / analytics migrated
- [ ] Spot-check top 20 URLs (homepage, services, book-call, top blogs, sample cities)

---

## Tools for validation

| Tool | Use |
|------|-----|
| View source | Raw `<title>`, canonical, meta description |
| `npm run build` + `npm run preview` | Production-like HTML |
| [Google Search Console](https://search.google.com/search-console) | Indexing, sitemaps, canonical issues |
| [Rich Results Test](https://search.google.com/test/rich-results) | JSON-LD |
| [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/) | OG tags |
| Screaming Frog / Sitebulb (optional) | Crawl staging or preview build |

---

## What is out of scope today

- Automatic meta from a CMS (content is in `.astro` files)
- hreflang / multi-language (some content mentions bilingual SEO; no hreflang tags)
- Automatic `Article` schema on all posts
- CI SEO checks (no automated title/canonical lint)
- PR preview `robots` noindex (not in deploy workflow)

---

## Related documentation

| Document | Topics |
|----------|--------|
| [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) | Writing pages, canonical examples |
| [DEPLOYMENT.md](./DEPLOYMENT.md) | Staging vs production cutover |
| [TEMPLATES.md](./TEMPLATES.md) | Which props each template accepts |
| [SITE-OVERVIEW.md](./SITE-OVERVIEW.md) | Site scale and architecture |
