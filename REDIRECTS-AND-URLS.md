# Redirects and URLs

This is the cheat sheet for **which URL is “real”** on goconstellation.com, where the Astro file actually lives, and what still needs a redirect before or at launch.

Production today is WordPress. Staging is Astro on Cloudflare (`con-staging`). Those two often use **different paths for the same content**—that’s normal for this migration, but it only works if redirects and canonicals agree.

**Related:** [SEO.md](./SEO.md) · [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) · [MIGRATION-TRACKER.md](./MIGRATION-TRACKER.md) · [TODO.md](./TODO.md) · [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md)

---

## The basic idea

In Astro, the URL comes from the file path:

- `src/pages/foo.astro` → `/foo/`
- `src/pages/blog/bar.astro` → `/blog/bar/`
- `src/pages/case-study/baz.astro` → `/case-study/baz/`

The **`canonicalUrl`** in each page is what you tell Google is the official URL. It often still points at the **old WordPress path** on purpose (SEO history, backlinks, Search Console).

So you can have:

| What | Example |
|------|---------|
| File | `src/pages/blog/law-firm-seo.astro` |
| Staging browser URL | `https://staging…/blog/law-firm-seo/` |
| Canonical / production URL | `https://goconstellation.com/law-firm-seo/` |

That’s fine **if** production either serves the page at the canonical path or **301s** there. If you only set the canonical and never redirect, you split traffic between two URLs.

**Rule we’ve been following:** pick the production URL that should rank, set `canonicalUrl` to that, then make Astro’s public path match **or** add a redirect.

---

## Site host and trailing slashes

- `astro.config.mjs` sets `site: 'https://www.goconstellation.com'`.
- Most production URLs use a **trailing slash** (`/about-us/` not `/about-us`).
- Decide **www vs non-www** once with whoever owns DNS, then make canonicals, redirects, and GSC property match. Don’t mix `www.goconstellation.com` and `goconstellation.com` on different pages.

---

## Redirects already in the repo

These six entries live in `astro.config.mjs` today (three slug pairs, with and without trailing slash):

| Old URL (WordPress-style long slug) | New URL |
|-------------------------------------|---------|
| `/mass-tort-attorney-marketing-strategies-for-client-growth/` | `/mass-tort-attorney-marketing/` |
| `/mastering-employment-lawyer-marketing-strategies-for-success/` | `/employment-lawyer-marketing/` |
| `/maximizing-success-in-workers-comp-lawyer-marketing-strategies-and-insights/` | `/workers-comp-lawyer-marketing/` |

That’s it for in-repo redirects. Everything else from the old site still needs to be mapped—either here or in **Cloudflare bulk redirects** at cutover.

---

## Internal links that don’t match Astro paths

Some nav links point at URLs that **don’t have a matching `.astro` file** at that path (or the content moved under `/blog/`).

### Footer (`SiteFooter.astro`)

| Link in footer | What Astro actually has | Fix |
|----------------|-------------------------|-----|
| `/lawyer-advertising/` | `src/pages/blog/lawyer-advertising.astro` → **`/blog/lawyer-advertising/`** | Redirect root → blog, or change link |
| `/law-website-content/` | `src/pages/blog/law-website-content.astro` → **`/blog/law-website-content/`** | Same |
| `/marketing-ideas-law-firms/` | `src/pages/blog/marketing-ideas-law-firms.astro` → **`/blog/…`** | Same |
| `/sitemap_index.xml` | WordPress sitemap — **not** Astro | Point to new `/sitemap.xml` when it exists |

### Header (`SiteHeader.astro`)

Header is mostly aligned: Advertising goes to `/law-firm-ppc/`, which exists.

### Services hub (`services.astro`)

| Link on page | Problem |
|--------------|---------|
| `/advertising/` | **No page.** PPC content is at `/law-firm-ppc/` (canonical on that file is `/ppc-services-lawyers/` — see below) |

Secondary service cards (`/law-firm-branding/`, `/meta-ads/`, etc.) match real files.

---

## Same page, different paths (canonical vs file)

These are built and work on staging, but the **canonical** still names another production URL. At launch you usually want a **301 from canonical path → Astro path** (or move the file—redirect is easier).

| Astro path (what staging serves) | `canonicalUrl` points at |
|----------------------------------|---------------------------|
| `/law-firm-ppc/` | `https://goconstellation.com/ppc-services-lawyers/` |

So bookmarks and ads may still use `/ppc-services-lawyers/` on WordPress. Plan a redirect to `/law-firm-ppc/` **or** rename the file to match production—team call.

---

## Production URLs that may still need a page or redirect

From the live WordPress page sitemap and nav—not all have an Astro file at the **same** path yet. Treat this as a working list; confirm against the [migration sheet](https://docs.google.com/spreadsheets/d/13jFsEzuXJp5MbTvwvQ1JmuTFIeLlGJ3zLXCSAUlMf4A/edit) before launch.

| Production URL | Notes |
|----------------|--------|
| `/ppc-services-lawyers/` | Content on `/law-firm-ppc/` — redirect |
| `/advertising/` | Linked from `services.astro` — redirect to PPC or build page |
| `/legal-marketing-services/` | No matching top-level `.astro` |
| `/elite-legal-marketing-services/` | No matching top-level `.astro` |
| `/law-firm-website-design-services/` | No matching top-level `.astro` (web design is `/law-firm-web-design/`) |
| `/law-firm-seo/` | Hub on WP; Astro has `/law-firm-seo/*` guides but no index at `/law-firm-seo/` |
| `/lawyer-advertising/` | File under `/blog/lawyer-advertising/` |

Export the **Service Pages** and **Generic** tabs from the sheet and append rows here as you close them.

---

## Blog posts at the root vs under `/blog/`

A lot of WordPress posts ranked at the **root** (`/law-firm-seo/`, `/lawyer-advertising/`). In Astro many of those files sit in `src/pages/blog/`, so staging shows `/blog/...`.

**What to do for each post on the sheet:**

1. Check `canonicalUrl` in the `.astro` file.
2. If canonical is `https://goconstellation.com/some-slug/` and the file is `src/pages/blog/some-slug.astro`, add:

   ```js
   // astro.config.mjs redirects — example
   '/some-slug/': '/blog/some-slug/',
   '/some-slug': '/blog/some-slug/',
   ```

   **Or** the other direction if you decide the `/blog/` URL should be canonical—just don’t leave both live without a redirect.

Grep tip:

```bash
grep -r 'canonicalUrl="https://goconstellation.com/' src/pages/blog/ | head
```

Compare each canonical path to `src/pages/...` path.

---

## Case studies: folder changed, slugs didn’t

Case studies in Astro live under `/case-study/<short-slug>/`, but many `canonicalUrl` values still use the **old WordPress slug** (often longer, sometimes at the root).

Examples:

| Astro URL | Canonical (production) |
|-----------|-------------------------|
| `/case-study/kairos-law-group/` | `/kairos-law-group-40xroi-case-study/` |
| `/case-study/sabbeth-law/` | `/seo-growth-case-study-sabbeth-law/` |
| `/case-study/cheryl-davis-law/` | `/bankruptcy-lawyer-seo-case-study/` |

For launch, each of those old URLs needs a **301 to the new `/case-study/...` path** (unless you intentionally keep the old URL as canonical and move the file—which we generally didn’t).

Internal links in blog HTML often still point at the **old** URLs too; a crawl before cutover will surface hundreds of those.

---

## Podcasts

Same pattern as blogs: file at `/podcasts/<slug>/`, canonical sometimes at root (`/how-to-close-more-cases/` etc.). Same fix: redirect old → new or align canonical with the path you actually serve.

---

## City pages

Usually straightforward: `attorney-marketing-austin.astro` → `/attorney-marketing-austin/` and canonical matches. Fewer path surprises than blog/case study. Still verify the 118 rows on the sheet—slug typos (`atlanta-ga` vs `atlanta`) break redirects.

---

## Where to put redirects at launch

| Method | Good for |
|--------|----------|
| **`astro.config.mjs` `redirects`** | Hundreds of stable rules in git; Astro emits them at build time |
| **Cloudflare Bulk Redirects** | Huge lists, quick changes without redeploy; good for one-off legacy WP URLs |
| **Both** | Common: common patterns in Astro, long tail in Cloudflare |

Test on staging after deploy: `curl -I https://staging.goconstellation.com/old-path/` and look for `301` and correct `Location`.

---

## How to build the full redirect list

1. Export URL column from each tab of the [migration tracker](https://docs.google.com/spreadsheets/d/13jFsEzuXJp5MbTvwvQ1JmuTFIeLlGJ3zLXCSAUlMf4A/edit) (see `documents/data/README.md`).
2. For each URL, note **production path** vs **Astro path** (from file location + `npm run build` sitemap when you have one).
3. Flag rows where they differ → candidate redirect.
4. Pull top URLs from Google Search Console and run through the same check.
5. Crawl staging or `dist/` with a link checker; fix internal links where you can, redirect the rest.

---

## `/examples/*` routes

Ten template preview URLs under `/examples/…` ship in production builds today. They’re for QA/Backstop, not for the public site. Before launch: **noindex**, drop from sitemap, or exclude from build—don’t rely on redirects alone unless you want them gone from indexes.

---

## Maintenance

When you add or move a page:

1. Set `canonicalUrl` to the URL you want indexed.
2. If the file path ≠ that URL, add a redirect (and update this doc or the sheet).
3. Update header/footer links so you’re not creating new 404s.
4. Re-run a quick link check on staging after merge to `main`.

---

## Quick reference

| Question | Answer |
|----------|--------|
| What controls the staging URL? | File path under `src/pages/` |
| What controls Google’s preferred URL? | `canonicalUrl` in the page / template |
| Where are redirects in git? | `astro.config.mjs` → `redirects` |
| How many redirects in git today? | 3 slug pairs (6 rules with slashes) |
| Footer sitemap link | Still WordPress — change at launch |
| Sheet for all manifest URLs | Google Sheet + [MIGRATION-TRACKER.md](./MIGRATION-TRACKER.md) |

This doc is meant to grow. When you add a redirect rule, add a line to the tables above so the next person isn’t guessing.
