# Content Guide

This guide explains **how content lives in the repo**, how to add or change it by hand, and how bulk migration scripts fit in. It is written for editors, marketers, and developers who need to ship a page without guessing where things go.

For template props and slots, see [TEMPLATES.md](./TEMPLATES.md). For commands and local setup, see [DEVELOPMENT.md](./DEVELOPMENT.md).

---

## The one rule to remember

**Content is not in a CMS.** It lives in `.astro` files under `src/pages/` (and images under `public/images/`). To change a paragraph, you edit Git. To publish, you merge to `main` and CI builds static HTML.

There is no “save draft” in WordPress; only branches and pull requests.

---

## How a page is structured

Every content page is usually two parts:

1. **Frontmatter block** (between `---` lines): imports the template and sets SEO props.
2. **Body**: either passed into the template as slot HTML, or absent when the template owns all markup (city pages, homepage).

```astro
---
import BlogPostTemplate from '../../layouts/templates/BlogPostTemplate.astro';
---

<BlogPostTemplate
  seoTitle="Your Page Title"
  seoDescription="Meta description for search and social."
  canonicalUrl="https://goconstellation.com/your-url/"
  publishDate="2026-05-18"
>
  <!-- Article HTML goes here -->
  <h2>Section heading</h2>
  <p>Paragraph text.</p>
</BlogPostTemplate>
```

The **URL** comes from the file path, not from `canonicalUrl`. See [URLs vs canonicals](#urls-vs-canonicals) below. That distinction trips people up during migration.

---

## Content types at a glance

| Type | Typical path | Template | Body location |
|------|--------------|----------|---------------|
| Blog post | `src/pages/blog/<slug>.astro` | `BlogPostTemplate` | Slot (HTML) |
| Nested article | `src/pages/law-firm-seo/<slug>.astro` | `BlogPostTemplate` | Slot (HTML) |
| Practice area guide | `src/pages/<practice>-marketing.astro` | `PracticeAreaTemplate` | Slot (HTML) |
| City landing page | `src/pages/attorney-marketing-<city>.astro` | `CityTemplate` | Inside template (props only) |
| Case study | `src/pages/case-study/<slug>.astro` | `CaseStudyNarrativeTemplate` | Named slots `before` / `after` |
| Podcast episode | `src/pages/podcasts/<slug>.astro` | `PodcastTemplate` | Slot (show notes) |
| Agency review | `src/pages/blog/<agency>-review.astro` | `AgencyReviewTemplate` | Props + optional slot |
| Pillar guide | `src/pages/blog/<guide>.astro` | `GuidesTemplate` | Slot; `chapters` use `id` not `anchor` |
| Service page (simple) | `src/pages/meta-ads.astro` | `ServiceTemplate` | Mostly props |
| Legal / utility | `src/pages/privacy-policy.astro` | `CoreTemplate` | Slot |
| Custom flagship | `book-call`, `law-firm-seo-services`, etc. | None | Full custom markup |

---

## URLs vs canonicals

| Concept | Controlled by | Example |
|---------|---------------|---------|
| **Staging/local URL** | File path under `src/pages/` | `src/pages/blog/law-firm-seo.astro` → `/blog/law-firm-seo/` |
| **Canonical URL** | `canonicalUrl` prop (and production redirects) | May point elsewhere for SEO continuity |

During the WordPress migration, many production URLs **do not match** the Astro file path. The canonical tells Google which URL is authoritative.

**Example:** `src/pages/blog/law-firm-seo.astro` is served at `/blog/law-firm-seo/` locally, but its canonical may be `https://goconstellation.com/law-firm-seo/` because that is where the live site has always ranked.

**When adding a page:**

1. Decide the **production URL** you want to keep (or create).
2. Place the file so Astro’s path matches **or** add a redirect in `astro.config.mjs` if paths differ.
3. Set `canonicalUrl` to the production URL you want indexed.

If you change a production URL, add a redirect in `astro.config.mjs`; do not only change the canonical.

---

## SEO and metadata conventions

Authoring conventions below. For the full metadata pipeline, redirects, sitemaps, and `robots.txt`, see [SEO.md](./SEO.md). For production cutover, see [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md).

### Title (`seoTitle`)

- Used for `<title>` and social share titles.
- Many pages use: `Primary Keyword Phrase | Constellation Marketing`
- Shorter titles are fine for well-known pages (e.g. `Winning Law Firm SEO: The Definitive Guide` without the suffix).
- Keep under ~60 characters when possible; Google truncates longer titles.

### Meta description (`seoDescription`)

- One or two sentences; benefit-led, includes primary keyword naturally.
- Aim for ~150–160 characters; not a hard limit.
- Every indexable page should have one.

### Canonical (`canonicalUrl`)

- Full absolute URL: `https://goconstellation.com/...`
- Trailing slash should match production convention (most URLs use a trailing `/`).
- Use `https://goconstellation.com` (not `www`) unless the team standardizes otherwise; match existing pages on the same section.

### Open Graph image (`ogImage`)

- Optional; defaults to `/images/constellation-og.jpg` via `BaseLayout`.
- Set for high-value pages: `ogImage="/images/your-hero.webp"`

### Dates

- Blog / podcast / guides: `publishDate="2026-05-18"` (ISO `YYYY-MM-DD`).
- Display formatting is handled in templates.

---

## Adding a new blog post (manual)

### 1. Choose the slug and path

- Standard post: `src/pages/blog/my-new-post.astro` → `/blog/my-new-post/`
- Nested topic: `src/pages/law-firm-seo/my-topic.astro` → `/law-firm-seo/my-topic/`

Check production sitemap or Search Console if replacing an existing WordPress URL.

### 2. Create the file

Copy a similar post or `src/pages/examples/blog-post-example.astro`.

```astro
---
import BlogPostTemplate from '../../layouts/templates/BlogPostTemplate.astro';
---

<BlogPostTemplate
  seoTitle="Your Headline | Constellation Marketing"
  seoDescription="One-sentence summary for search results."
  canonicalUrl="https://goconstellation.com/blog/your-slug/"
  publishDate="2026-05-18"
  chapters={[
    { label: "First Section", anchor: "first-section" },
    { label: "Second Section", anchor: "second-section" },
  ]}
>

<h2 id="first-section">First Section</h2>
<p>Opening paragraph…</p>

<h2 id="second-section">Second Section</h2>
<p>More content…</p>

</BlogPostTemplate>
```

**Import path:** Two levels up from `blog/` (`../../`). Three levels from `blog/nested/` if you ever nest deeper.

### 3. Write the body as HTML

Use semantic tags: `<h2>`, `<h3>`, `<p>`, `<ul>`, `<ol>`, `<blockquote>`, `<a>`, `<img>`.

The template styles `.article-body`; you do not need inline styles on every paragraph.

### 4. Table of contents (`chapters`)

Each chapter needs:

- `label`: text shown in the sidebar TOC
- `anchor`: must match an `id` on a heading in the body

```astro
chapters={[
  { label: "Why It Matters", anchor: "why-it-matters" },
]}
```

```html
<h2 id="why-it-matters">Why It Matters</h2>
```

Cap at ~10–12 chapters for readability; migrate scripts cap at 12 automatically.

### 5. Images

```html
<img
  src="/images/wp-uploads/2026/05/my-diagram.webp"
  alt="Descriptive alt text for accessibility and SEO"
  width="750"
  height="457"
  loading="lazy"
/>
```

- Put files in `public/images/` (often `public/images/wp-uploads/...` for migrated media).
- Prefer **WebP** for photos; keep filenames lowercase with hyphens.
- Always set meaningful `alt` text.

### 6. Internal links

Prefer **root-relative** paths so links work on staging and production:

```html
<a href="/book-call/">Book a call</a>
<a href="/law-firm-seo-services/">Law firm SEO services</a>
<a href="/case-study/sabbeth-law/">Sabbeth Law case study</a>
```

Migrated posts often still use full `https://goconstellation.com/...` URLs. Those work, but root-relative links are easier to maintain.

### 7. Verify

```bash
npm run dev
# Open http://localhost:4321/blog/your-slug/
npm run build
```

---

## Adding a practice area guide

Used for top-level pages like `/criminal-defense-marketing/`, `/immigration-law-firm-marketing/`.

1. Create `src/pages/your-practice-marketing.astro`.
2. Use `PracticeAreaTemplate` (see `src/pages/immigration-law-firm-marketing.astro` or `/examples/practice-area-example/`).
3. Pass `practiceArea`, `chapters`, and optional `caseStudies` array for sidebar proof cards.
4. Use `id` on `<h2>` headings matching chapter anchors.

**Helpful CSS classes** in the slot (defined in the template):

| Class | Purpose |
|-------|---------|
| `stat-callout` | Grid of stat boxes |
| `cta-inset` | Green call-to-action block with button |

```html
<div class="cta-inset">
  <h3>Ready to grow your practice?</h3>
  <p>Short pitch.</p>
  <a href="/book-call/">Schedule a Free Consultation</a>
</div>
```

---

## Adding a city landing page

City pages are mostly **configuration**, not long HTML.

1. Create `src/pages/attorney-marketing-austin.astro` (or `law-firm-marketing-austin.astro`).
2. Use `CityTemplate` with city-specific images in `public/images/`.

```astro
---
import CityTemplate from '../layouts/templates/CityTemplate.astro';
---

<CityTemplate
  seoTitle="Attorney Marketing in Austin | Constellation Marketing"
  seoDescription="Short meta description mentioning Austin and Texas."
  canonicalUrl="https://goconstellation.com/attorney-marketing-austin/"
  city="Austin"
  state="Texas"
  heroImage="/images/attorney-marketing-austin-title.webp"
  heroSubtitle="Your Legal Marketing Partner for Dominating Local Search in Austin, Texas"
  landscapeImage="/images/austin-building.webp"
/>
```

**Images:** Add WebP hero and landscape files before committing. Naming pattern: `attorney-marketing-{city}-title.webp`, `{city}-building.webp`.

**Note:** Bulk-generated pages may include `serviceImages={[...]}`. The template does not use that prop yet; update `heroImage` / `landscapeImage` for visible changes.

**Bulk creation:** `scripts/build_city_pages.py` scrapes production, downloads images, writes `.astro` files. Update hardcoded paths inside the script for your machine before running. Skips cities that already have a file.

---

## Adding a case study

1. Create `src/pages/case-study/client-slug.astro`.
2. Use `CaseStudyNarrativeTemplate`.
3. Fill structured props (`clientName`, `keyResult`, `beforePoints`, `afterPoints`, etc.).
4. Put narrative prose in named slots:

```astro
<CaseStudyNarrativeTemplate
  seoTitle="..."
  canonicalUrl="https://goconstellation.com/case-study/client-slug/"
  clientName="Firm Name"
  pageTitle="One-line story headline"
  keyResult="Headline metric"
  ...
>
  <div slot="before">
    <p>Story before working with Constellation…</p>
  </div>
  <div slot="after">
    <p>What we did and results…</p>
  </div>
</CaseStudyNarrativeTemplate>
```

Add the study to `src/pages/case-studies.astro` if it should appear in the index grid (that page is custom-built, not automatic).

---

## Adding a podcast episode

1. Create `src/pages/podcasts/episode-slug.astro`.
2. Use `PodcastTemplate` with `embedUrl` (YouTube embed URL), platform links, `duration`, `publishDate`.
3. Put show notes in the default slot.

Check whether production uses `/podcasts/episode-slug/` or a root URL; set `canonicalUrl` to match production.

---

## Adding an agency review

Use `AgencyReviewTemplate` with structured agency metadata and `comparisonRows`. Put long-form sections (pricing deep-dive, verdict) in the default slot.

Reference: `src/pages/blog/rankings-io-review.astro`.

---

## Editing existing migrated content

Most blog content was scraped from WordPress. Expect artifacts:

| Artifact | What to do |
|----------|------------|
| `&#8217;` and other entities | Replace with `'` or `"` for readability, or leave (browsers render correctly) |
| `data-start` / `data-color` on `<p>` | Safe to remove; Divi/editor leftovers |
| Empty `<ol></ol>` | Delete |
| Broken iframes / `mhtml.blink` embeds | Replace with proper YouTube embed or remove |
| `class="wp-image-249016 aligncenter"` | Optional cleanup; images still work |
| Full `https://goconstellation.com` links | Optionally convert to `/path/` relative links |
| Duplicate H1 in body | Remove; template provides page title |

**Do not** bulk find-replace across all posts without review; some “messy” HTML may be intentional for spacing or embeds.

---

## Images and media

### Where files go

```
public/images/
├── wp-uploads/          # Migrated WordPress media (by year/month)
├── case-study/          # Case study heroes
├── attorney-marketing-* # City heroes and scenery
└── constellation-og.jpg # Default social image
```

### URL pattern

File: `public/images/foo.webp` → use `src="/images/foo.webp"` in HTML.

### Optimization

- Use WebP for photos; PNG for logos with transparency.
- Resize wide images before commit (city script uses max width 1600px).
- Avoid committing multi-megabyte originals.

### Favicon and global assets

- Favicon: `public/images/cropped-Constellation-FavIcon-32x32.png` (referenced in `BaseLayout`).
- Fonts: `public/fonts/poppins-*.woff2`

---

## Lists and indexes (manual maintenance)

Some listing pages are **hand-curated arrays**, not auto-generated from the filesystem:

| Page | What to update |
|------|----------------|
| `src/pages/blog/index.astro` | `posts` array: title, `href`, date |
| `src/pages/podcasts/index.astro` | Episode list |
| `src/pages/case-studies.astro` | Case study cards / filters |
| `src/components/SiteHeader.astro` | Nav links for practice areas and services |

Adding a blog file does **not** automatically add it to `/blog/` index; you must add a row if it should appear there.

---

## Redirects and URL changes

Legacy URL changes belong in `astro.config.mjs`:

```js
redirects: {
  '/old-slug': '/new-slug',
  '/old-slug/': '/new-slug/',
},
```

Add **both** with and without trailing slash if production used both.

---

## Bulk migration scripts

Use scripts when migrating many pages from production, not for routine copy edits.

See **[MIGRATION-TOOLING.md](./MIGRATION-TOOLING.md)** for full documentation on each script, commands, overwrite behavior, Backstop, Refore, and troubleshooting.

---

## Content that lives outside page files

| Item | Location |
|------|----------|
| Global nav links | `src/components/SiteHeader.astro` |
| Footer links | `src/components/SiteFooter.astro` |
| Homepage sections | `src/layouts/templates/HomepageTemplate.astro` |
| City page body | `src/layouts/templates/CityTemplate.astro` |
| Sitewide Divi CSS overrides | `src/styles/divi-theme-options.css` |
| Legacy Divi stylesheet | `public/css/production-extracted.css` |

Changing “something on every page” usually means header, footer, or `BaseLayout`, not one blog file.

---

## Publishing workflow

1. Edit content on a branch.
2. `npm run dev`: spot-check pages you touched.
3. `npm run build`: fix any Astro/HTML errors (unclosed tags break the build).
4. Open PR; review diff (large HTML files create noisy diffs, which is normal).
5. Merge to `main` → GitHub Actions deploys to Cloudflare `con-staging`.
6. Verify on staging URL; compare critical URLs to production if SEO-sensitive.

There is no separate “publish” button; merge is publish.

---

## Quality checklist (before merge)

- [ ] `seoTitle` and `seoDescription` set
- [ ] `canonicalUrl` matches intended production URL
- [ ] File path and canonical aligned, or redirect added
- [ ] Heading hierarchy: one logical H1 (from template); body starts at H2
- [ ] Chapter anchors match `id` attributes (if using TOC)
- [ ] Images exist in `public/`, have `alt` text
- [ ] Internal links work (click in preview)
- [ ] `npm run build` passes
- [ ] New blog post added to `blog/index.astro` if it should be listed

---

## Common questions

**Can I use Markdown?**  
Not by default. Content is HTML inside `.astro` files. Markdown would require adding MDX support; out of scope today.

**How do I unpublish a page?**  
Remove the file or stop linking to it; add a redirect if the URL was public. Static sites do not have a “draft” flag.

**Why does my post look fine locally but wrong on staging?**  
Check cached CSS, missing images (case-sensitive paths on Linux), or Divi class dependencies. Hard-refresh the browser.

**Should I edit production WordPress and the repo?**  
During migration, treat the **repo as the source of truth** for the Astro site. WordPress may remain live until cutover; coordinate which side is authoritative.

---

## Related documentation

| Document | Topics |
|----------|--------|
| [TEMPLATES.md](./TEMPLATES.md) | Template props, slots, which template to pick |
| [SEO.md](./SEO.md) | Canonicals, redirects, sitemaps, structured data |
| [DEVELOPMENT.md](./DEVELOPMENT.md) | npm commands, troubleshooting, Backstop |
| [SITE-OVERVIEW.md](./SITE-OVERVIEW.md) | Architecture and scale |
| [REFORE-APPROACH.md](../REFORE-APPROACH.md) | Homepage HTML migration |
