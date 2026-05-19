# Routing reference

How URLs work in this Astro static site: file paths, build output, canonicals, redirects, and where each content type lives.

**Use this doc when:** you need to know what URL a file produces, where to put a new page, or how staging paths relate to production SEO.

**Related:** [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md) (migration redirect map) · [NAVIGATION.md](./NAVIGATION.md) (header/footer links) · [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) (authoring) · [TEMPLATES.md](./TEMPLATES.md) (layouts) · [SEO.md](./SEO.md) (metadata)

---

## Three layers (don’t mix them up)

Every page involves three different “URLs.” They are often the same on city pages; they often **differ** on blogs and case studies.

| Layer | Controlled by | Example |
|-------|---------------|---------|
| **Astro route** (what the file builds) | Path under `src/pages/` | `src/pages/blog/law-firm-seo.astro` → `/blog/law-firm-seo/` |
| **Canonical** (what Google should index) | `canonicalUrl` prop → `<link rel="canonical">` in `BaseLayout` | `https://goconstellation.com/law-firm-seo/` |
| **Production history** (WordPress, backlinks, ads) | Old CMS paths; may need **301 redirects** | Same as canonical until you intentionally change it |

```
src/pages/blog/foo.astro
        │
        ▼  build
   /blog/foo/          ← browser on staging
        │
        ├─ canonicalUrl → /foo/     ← meta tag (may differ)
        │
        └─ redirect (if configured) → /foo/   ← HTTP 301 at launch
```

**Default if you omit `canonicalUrl`:** `BaseLayout` falls back to `Astro.url.href` (the URL you actually requested). That is fine for new pages where the file path **is** the production URL.

**Rule:** Pick the URL you want indexed, set `canonicalUrl` to that full absolute URL, then either match the file path or add a redirect. Details and tables of mismatches: [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md).

---

## Global config

From `astro.config.mjs`:

| Setting | Value | Effect |
|---------|-------|--------|
| `output` | `'static'` | Every route becomes a pre-rendered HTML file in `dist/` at build time |
| `site` | `https://www.goconstellation.com` | Base for absolute URLs in feeds/sitemaps when Astro generates them |
| `redirects` | 3 slug pairs (6 rules) | Build-time redirects; see [Redirects](#redirects) |

There is **no** `trailingSlash` option in config today. Astro static output uses **directory-style** URLs (`dist/blog/foo/index.html` → `/blog/foo/`). Production and internal links conventionally use a **trailing slash**.

**www vs apex:** Config uses `www`. Many `canonicalUrl` values use `https://goconstellation.com` without `www`. Standardize with DNS/Search Console before launch—see [SEO.md](./SEO.md).

---

## File-based routing rules

Astro maps `src/pages/` to site paths:

| File | URL |
|------|-----|
| `src/pages/index.astro` | `/` |
| `src/pages/about-us.astro` | `/about-us/` |
| `src/pages/blog/index.astro` | `/blog/` |
| `src/pages/blog/my-post.astro` | `/blog/my-post/` |
| `src/pages/law-firm-seo/on-page-optimization.astro` | `/law-firm-seo/on-page-optimization/` |
| `src/pages/case-study/kairos-law-group.astro` | `/case-study/kairos-law-group/` |

**Rules:**

- Filename (without `.astro`) = URL segment. Use **kebab-case** slugs.
- Folders create path prefixes. No `[]` dynamic routes in this repo—each URL is its own file.
- There is no `src/pages/[slug].astro` catch-all; ~386 production routes are explicit files.

**Import depth** grows with folder nesting—see [TEMPLATES.md](./TEMPLATES.md#import-paths-easy-to-get-wrong).

---

## Route inventory (~396 files)

Counts from `src/pages/**/*.astro` (approximate; run `find` after large merges):

| Segment | Count | URL pattern | Typical template |
|---------|------:|-------------|------------------|
| **Root** (`src/pages/*.astro`) | ~157 | `/<slug>/` | Mixed: `CityTemplate`, `PracticeAreaTemplate`, `ServiceTemplate`, custom |
| **`blog/`** | ~161 | `/blog/<slug>/` | `BlogPostTemplate`, `GuidesTemplate`, `AgencyReviewTemplate` |
| **City pages** (subset of root) | ~132 | `/attorney-marketing-<city>/` or `/law-firm-marketing-<city>/` | `CityTemplate` |
| **`case-study/`** | ~27 | `/case-study/<slug>/` | `CaseStudyNarrativeTemplate` |
| **`podcasts/`** | ~24 | `/podcasts/<slug>/` | `PodcastTemplate` |
| **`law-firm-seo/`** | ~13 | `/law-firm-seo/<slug>/` | `BlogPostTemplate` (nested guides) |
| **`lawyer-advertising/`** | ~3 | `/lawyer-advertising/<slug>/` | `BlogPostTemplate` / long-form |
| **`examples/`** | ~10 | `/examples/<name>/` | Template previews (QA only) |
| **Index routes** | 3 | `/`, `/blog/`, `/podcasts/` | Homepage, blog hub, podcast hub |

**Production routes:** ~386 pages if you exclude `src/pages/examples/`.

**Hub pages (not in a subfolder):**

| File | URL | Role |
|------|-----|------|
| `index.astro` | `/` | Homepage |
| `case-studies.astro` | `/case-studies/` | Case study index (links to `/case-study/...`) |
| `services.astro` | `/services/` | Services overview |
| `blog/index.astro` | `/blog/` | Blog index |
| `podcasts/index.astro` | `/podcasts/` | Podcast index |

Note: **index** is `/case-studies/` (plural); **detail** is `/case-study/<slug>/` (singular). WordPress often used yet another slug for the same story—redirects required.

---

## Route taxonomy (where to put a new page)

| Content type | Put the file here | URL you get | Template |
|--------------|-------------------|-------------|----------|
| New blog article | `src/pages/blog/<slug>.astro` | `/blog/<slug>/` | `BlogPostTemplate` |
| SEO guide chapter | `src/pages/law-firm-seo/<slug>.astro` | `/law-firm-seo/<slug>/` | `BlogPostTemplate` |
| City landing page | `src/pages/attorney-marketing-<city>.astro` or `law-firm-marketing-<city>.astro` | Same as filename | `CityTemplate` |
| Practice area pillar | `src/pages/<practice>-marketing.astro` | `/<practice>-marketing/` | `PracticeAreaTemplate` |
| Service marketing page | `src/pages/<service>.astro` (e.g. `meta-ads.astro`) | `/<service>/` | `ServiceTemplate` or custom |
| Case study | `src/pages/case-study/<slug>.astro` | `/case-study/<slug>/` | `CaseStudyNarrativeTemplate` |
| Podcast episode | `src/pages/podcasts/<slug>.astro` | `/podcasts/<slug>/` | `PodcastTemplate` |
| Legal / utility | `src/pages/privacy-policy.astro`, etc. | `/<slug>/` | `CoreTemplate` |
| Flagship custom | `book-call.astro`, `law-firm-seo-services.astro` | Root slug | `BaseLayout` + components |
| Template preview | `src/pages/examples/<name>.astro` | `/examples/<name>/` | Any (for QA) |

**City naming conventions:**

- `attorney-marketing-{city}` — e.g. `attorney-marketing-denver.astro` → `/attorney-marketing-denver/`
- `law-firm-marketing-{city}` — e.g. `law-firm-marketing-dallas.astro` → `/law-firm-marketing-dallas/`

Match the slug in the [migration tracker](https://docs.google.com/spreadsheets/d/13jFsEzuXJp5MbTvwvQ1JmuTFIeLlGJ3zLXCSAUlMf4A/edit) exactly (`atlanta-ga` vs `atlanta` breaks redirects).

**Practice areas in main nav** (header dropdown) — all root-level:

`/bankruptcy-law-firm-marketing/`, `/criminal-defense-marketing/`, `/estate-planning-marketing/`, `/family-law-marketing/`, `/immigration-law-firm-marketing/`, `/personal-injury-lawyer-marketing/`

**Services in main nav:**

| Nav label | Href | File |
|-----------|------|------|
| Law Firm SEO | `/law-firm-seo-services/` | `law-firm-seo-services.astro` |
| Advertising | `/law-firm-ppc/` | `law-firm-ppc.astro` |
| Website Design | `/law-firm-web-design/` | `law-firm-web-design.astro` |

Full nav audit: [NAVIGATION.md](./NAVIGATION.md).

---

## Canonical URL (`BaseLayout`)

`src/layouts/BaseLayout.astro` emits:

```html
<link rel="canonical" href={canonicalUrl} />
<meta property="og:url" content={canonicalUrl} />
```

Templates pass `canonicalUrl` from page frontmatter. Use a **full absolute URL** with the production host:

```astro
canonicalUrl="https://goconstellation.com/your-slug/"
```

**Known path vs canonical mismatch (fix with redirect or rename):**

| Astro route | `canonicalUrl` target |
|-------------|------------------------|
| `/law-firm-ppc/` | `https://goconstellation.com/ppc-services-lawyers/` |

Case studies: Astro path `/case-study/<short>/` vs canonical often at **root** with a long WordPress slug—see [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md#case-studies-folder-changed-slugs-didnt).

**CoreTemplate bug:** Some pages pass `title` instead of `seoTitle`; `CoreTemplate` expects `seoTitle` for `BaseLayout`. Affected pages may get wrong `<title>` until fixed—grep `CoreTemplate` usages.

---

## Redirects

### In `astro.config.mjs` (build-time)

Astro requires **both** slash variants for each pair:

```js
redirects: {
  '/old-slug': '/new-slug',
  '/old-slug/': '/new-slug/',
},
```

**Currently configured (3 pairs):**

| From | To |
|------|-----|
| `/mass-tort-attorney-marketing-strategies-for-client-growth/` | `/mass-tort-attorney-marketing/` |
| `/mastering-employment-lawyer-marketing-strategies-for-success/` | `/employment-lawyer-marketing/` |
| `/maximizing-success-in-workers-comp-lawyer-marketing-strategies-and-insights/` | `/workers-comp-lawyer-marketing/` |

### At launch (often hundreds more)

| Method | When to use |
|--------|-------------|
| `astro.config.mjs` | Stable rules in git; redeploy to change |
| **Cloudflare Bulk Redirects** | Long tail, quick edits without build |
| Both | Common split |

Test after deploy:

```bash
curl -I https://staging.goconstellation.com/old-path/
```

Look for `301` / `308` and correct `Location`.

Working list of path mismatches: [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md).

---

## Internal links vs routes

Prefer **relative paths** that match Astro routes (`/case-study/foo/`, `/blog/bar/`).

Legacy migrated HTML often still links to **WordPress URLs** (`https://goconstellation.com/bankruptcy-lawyer-seo-case-study/`). Those need redirects even if canonicals are correct.

**Footer paths that don’t match files today:**

| Linked URL | Actual Astro route |
|------------|-------------------|
| `/lawyer-advertising/` | `/blog/lawyer-advertising/` |
| `/law-website-content/` | `/blog/law-website-content/` |
| `/marketing-ideas-law-firms/` | `/blog/marketing-ideas-law-firms/` |

**Services hub:** `services.astro` links `/advertising/` — no page; PPC lives at `/law-firm-ppc/`.

See [NAVIGATION.md](./NAVIGATION.md).

---

## Build output

`npm run build` writes static HTML to `dist/`:

```
dist/
  index.html
  about-us/index.html
  blog/
    index.html
    some-post/index.html
  case-study/
    kairos-law-group/index.html
```

Cloudflare Pages serves `dist/` as-is. No Node server at request time.

**Preview locally:**

```bash
npm run build && npm run preview
# http://localhost:4321/<path>/
```

**Count routes:**

```bash
find src/pages -name '*.astro' | wc -l
find src/pages -name '*.astro' ! -path '*/examples/*' | wc -l
```

---

## Staging vs production routing

| Concern | Staging (`con-staging` on Cloudflare) | Production (after cutover) |
|---------|--------------------------------------|----------------------------|
| Host | `*.pages.dev` or `staging.goconstellation.com` | `www.goconstellation.com` |
| Auth | `functions/_middleware.ts` — HTTP Basic Auth | Remove or disable for public launch |
| `public/robots.txt` | `Disallow: /` (blocks crawlers) | Replace with allow + sitemap |
| Paths | Same as production build | Same file → same paths |
| Redirects | From `astro.config.mjs` + Cloudflare rules | Same |

Staging path **equals** production Astro path; only host and SEO files differ.

---

## `/examples/*` routes

Ten template preview pages under `src/pages/examples/`. They ship in production builds unless excluded.

**Before launch:** noindex, omit from sitemap, or remove from build—see [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md#examples-routes).

| Example URL | Purpose |
|-------------|---------|
| `/examples/blog-post-example/` | `BlogPostTemplate` |
| `/examples/city-example/` | `CityTemplate` |
| `/examples/homepage-example/` | `HomepageTemplate` |
| … | See `src/pages/examples/` |

---

## Adding or moving a route (checklist)

1. **Choose production URL** (sheet + Search Console + existing backlinks).
2. **Create** `src/pages/.../<slug>.astro` at the path you want to serve—or plan a redirect if the file must live elsewhere (e.g. blog under `/blog/` but canonical at root).
3. **Set** `canonicalUrl` to the production URL you want indexed.
4. **Add redirects** in `astro.config.mjs` (both slash variants) if file path ≠ canonical path.
5. **Update** `SiteHeader.astro` / `SiteFooter.astro` if the page belongs in nav.
6. **Fix** internal links in other `.astro` files and migrated HTML.
7. **Build** locally; hit the new path in `npm run preview`.
8. **Document** redirect in [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md) if non-obvious.
9. **Mark** row done in [migration tracker](https://docs.google.com/spreadsheets/d/13jFsEzuXJp5MbTvwvQ1JmuTFIeLlGJ3zLXCSAUlMf4A/edit).

---

## Common patterns from WordPress migration

### Blog at root vs `/blog/`

WordPress: `/law-firm-seo/`. Astro file: `src/pages/blog/law-firm-seo.astro` → `/blog/law-firm-seo/`.

**Options:**

- **A)** Redirect `/law-firm-seo/` → `/blog/law-firm-seo/` and update canonical to `/blog/...`
- **B)** Keep canonical at `/law-firm-seo/` and redirect root → blog (or move file to `src/pages/law-firm-seo.astro`)

Do not leave both URLs 200 without a redirect.

### Case study slug change

WordPress: `/kairos-law-group-40xroi-case-study/`. Astro: `/case-study/kairos-law-group/`.

301 old → new at launch; update in-content links over time.

### Podcast at root vs `/podcasts/`

Same pattern as blog. Check each episode’s `canonicalUrl`.

---

## Quick lookup

| Question | Answer |
|----------|--------|
| What URL does this file get? | Path under `src/pages/`, `.astro` → `/`, folders → `/` |
| What should Google index? | `canonicalUrl` in the page |
| Where are redirects? | `astro.config.mjs` → `redirects`; Cloudflare for bulk |
| How many routes? | ~396 total; ~386 excl. `/examples/` |
| Case study list URL | `/case-studies/` |
| Case study detail URL | `/case-study/<slug>/` |
| Blog index | `/blog/` |
| Podcast index | `/podcasts/` |
| Config site URL | `https://www.goconstellation.com` |
| Default canonical if omitted | Current request URL (`Astro.url.href`) |

---

## Related documentation

| Doc | Use for |
|-----|---------|
| [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md) | Production vs Astro paths, redirect backlog |
| [NAVIGATION.md](./NAVIGATION.md) | Header/footer hrefs |
| [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) | Authoring new pages |
| [TEMPLATES.md](./TEMPLATES.md) | Which layout to use |
| [MIGRATION-TRACKER.md](./MIGRATION-TRACKER.md) | Sheet tabs and URL manifest |
| [DEPLOYMENT.md](./DEPLOYMENT.md) | How `dist/` reaches Cloudflare |
| [SEO.md](./SEO.md) | Sitemap, robots, Search Console |
