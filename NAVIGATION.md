# Navigation

Where the site menu lives, what each link points at, and what’s broken or inconsistent. If you’re changing a URL or adding a page, start here before you touch `SiteHeader.astro` or `SiteFooter.astro`.

**Related:** [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md) · [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) · [TODO.md](./TODO.md)

---

## One place controls most of it

Global nav is **not** pulled from the migration Google Sheet. It’s hardcoded in two components:

| File | What it drives |
|------|----------------|
| `src/components/SiteHeader.astro` | Sticky green pill bar — logo, dropdowns, Book a Call |
| `src/components/SiteFooter.astro` | Dark footer — four columns + social |

Every page that uses `BaseLayout` (or a template that includes header/footer) gets the same links. City pages, blog posts, and service pages don’t define their own top nav—they inherit these.

To add or rename a menu item, edit the arrays at the top of `SiteHeader.astro` (`practiceAreas`, `services`, `resources`) or the footer markup in `SiteFooter.astro`.

---

## Header layout (desktop)

At **1024px and up**, the main nav shows. Below that you only get the hamburger—the desktop dropdowns are hidden (`display: none` on `.main-nav` until the media query).

```
[ Logo → / ]  [ Results ]  [ Practice Areas ▾ ]  [ Services ▾ ]  [ Resources ▾ ]  [ Book a Call ]
```

| Label | Href | Page exists? |
|-------|------|--------------|
| Logo | `/` | Yes — `index.astro` |
| Results | `/case-studies/` | Yes — `case-studies.astro` |
| Book a Call | `/book-call/` | Yes — `book-call.astro` |

### Practice Areas dropdown (6 items)

| Label | Href | File |
|-------|------|------|
| Bankruptcy | `/bankruptcy-law-firm-marketing/` | `bankruptcy-law-firm-marketing.astro` |
| Criminal Defense | `/criminal-defense-marketing/` | `criminal-defense-marketing.astro` |
| Estate Planning | `/estate-planning-marketing/` | `estate-planning-marketing.astro` |
| Family Law | `/family-law-marketing/` | `family-law-marketing.astro` |
| Immigration Law | `/immigration-law-firm-marketing/` | `immigration-law-firm-marketing.astro` |
| Personal Injury | `/personal-injury-lawyer-marketing/` | `personal-injury-lawyer-marketing.astro` |

All six work on staging.

**Not in the header** (but built in the repo): DUI, employment law, mass tort, medical malpractice, real estate, workers comp—each has its own `*-marketing.astro` page. They’re on the migration sheet as practice areas; they just never got added to the dropdown. Worth aligning with whoever owns IA.

### Services dropdown

| Label | Href | Page exists? |
|-------|------|--------------|
| **All Services** (highlighted) | `/services/` | Yes |
| Law Firm SEO | `/law-firm-seo-services/` | Yes |
| Advertising | `/law-firm-ppc/` | Yes (canonical on production may be `/ppc-services-lawyers/` — see redirects doc) |
| Website Design | `/law-firm-web-design/` | Yes |

**Not in the header dropdown** but live as pages: `/meta-ads/`, `/law-firm-branding/`, `/email-marketing-for-law-firms/`, `/social-media-for-law-firms/`, `/ai-chatbot-for-law-firms/`, `/bilingual-seo-law-firms/`. Several of those are linked from the **Services hub** page body instead.

### Resources dropdown

| Label | Href | Page exists? |
|-------|------|--------------|
| About Us | `/about-us/` | Yes |
| Our Approach | `/services/` | Yes (same as “All Services”) |
| Blog | `/blog/` | Yes — `blog/index.astro` |
| Podcast | `/podcasts/` | Yes — `podcasts/index.astro` |

---

## Footer layout

Four columns: brand + social, Practice Areas, Services, Company/Resources.

### Practice Areas column

Same six links as the header. All good.

### Services column

| Label | Href | Status |
|-------|------|--------|
| Law Firm SEO | `/law-firm-seo-services/` | OK |
| Law Firm Advertising | `/lawyer-advertising/` | **Broken path** — content is at `/blog/lawyer-advertising/` |
| Web Design | `/law-firm-web-design/` | OK |
| Website Content | `/law-website-content/` | **Broken path** — content is at `/blog/law-website-content/` |
| Marketing Ideas | `/marketing-ideas-law-firms/` | **Broken path** — content is at `/blog/marketing-ideas-law-firms/` |

Fix options: change the footer hrefs to `/blog/...`, or add redirects from the root paths (see [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md)).

### Company column

| Label | Href | Status |
|-------|------|--------|
| About Us | `/about-us/` | OK |
| Results | `/case-studies/` | OK |
| Blog | `/blog/` | OK |
| Podcast | `/podcasts/` | OK |
| Book a Consultation | `/book-call/` | OK |

### Resources column (footer)

| Label | Href | Status |
|-------|------|--------|
| Sitemap | `/sitemap_index.xml` | **WordPress** — will 404 on Astro until you ship `/sitemap.xml` and update the link |
| Privacy Policy | `/privacy-policy/` | OK |
| Terms of Service | `/terms-of-service/` | OK |
| Submit a Ticket | `/support-center/` | OK |
| FAQs | `/faq/` | OK |

### Social (external)

Facebook, Instagram, LinkedIn, YouTube, X, Reddit — all `target="_blank"` with `rel="noopener noreferrer"`. No Astro pages involved.

---

## Services hub page (`/services/`)

Separate from header/footer arrays: `src/pages/services.astro` has its own link list.

| Card | Href | Status |
|------|------|--------|
| SEO | `/law-firm-seo-services/` | OK |
| Advertising | `/advertising/` | **No page** — should probably be `/law-firm-ppc/` |
| Website Development | `/law-firm-web-design/` | OK |
| Branding, Email, Social, Bilingual SEO, AI Chatbot | various | OK — secondary grid |

So you’ve got **three different “advertising” URLs** in play: header → `/law-firm-ppc/`, footer → `/lawyer-advertising/` (blog), services hub → `/advertising/` (missing). Worth picking one primary URL and making the rest redirects.

---

## Mobile menu — known gap

There’s a hamburger below 1024px that toggles a `mobile-open` class on `.main-nav`, but **there’s no CSS** in `SiteHeader.astro` that shows the nav when `mobile-open` is set. On phones/tablets, users may only see the logo and menu icon with no way to open the full tree unless that’s fixed elsewhere (global CSS). Flag for QA on real devices.

Dropdowns on desktop use hover and `:focus-within`; there’s a small script for `aria-expanded` on click for keyboard users.

---

## Nav inside templates (not the global menu)

Templates repeat a few links in the page body—those are **not** controlled by `SiteHeader`/`SiteFooter`.

| Pattern | Typical links | Where |
|---------|---------------|--------|
| Back to hub | `← Blog` → `/blog/`, `← Podcast` → `/podcasts/`, `← Case Studies` → `/case-studies/` | Blog, podcast, case study templates |
| Primary CTA | `/book-call/` | Most templates, city pages, practice areas |
| City service blocks | SEO, PPC, web design, content, social | `CityTemplate.astro` |
| Case study cross-links | Other `/case-study/...` URLs | Frontmatter `relatedCases` |

### Homepage variants (watch for stale links)

`HomepageTemplate-v2.astro` and `v3` still reference paths like `/free-strategy-session/`, `/about/`, `/book-a-call` (no trailing slash). Production homepage should be `index.astro` → `HomepageTemplate` (v1). If v2/v3 are only for `/examples/`, fine; if they’re ever swapped in as live, those links need updating to `/book-call/` and `/about-us/`.

---

## Header vs footer — quick diff

| Topic | Header | Footer |
|-------|--------|--------|
| Advertising / PPC | `/law-firm-ppc/` | `/lawyer-advertising/` (wrong) |
| Results label | “Results” | “Results” |
| Extra service links | Only 3 + hub | Content + marketing ideas (wrong paths) |
| Legal / support | Not in header | Privacy, terms, support, FAQ |
| Sitemap | — | WordPress URL |

---

## What to check before launch

1. Click every header and footer link on **staging** (desktop and mobile).
2. Fix the three footer service links or add redirects.
3. Fix `/advertising/` on `services.astro`.
4. Point footer sitemap at the new Astro sitemap.
5. Decide whether to add the other six practice-area pages to the dropdown.
6. Decide whether secondary services (`meta-ads`, etc.) belong in the Services dropdown.
7. Fix or verify mobile nav actually opens.

A simple crawl from `/` with a link checker after deploy catches anything this doc missed.

---

## Changing nav safely

1. Edit `SiteHeader.astro` and/or `SiteFooter.astro`.
2. Grep the repo for the old path (`grep -r "/old-path/" src/`).
3. Update [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md) if production URLs change.
4. `npm run build` and click-test on preview.

The migration sheet lists **325** manifest URLs; the global nav only exposes a **small subset** on purpose. Don’t assume every built page is linked from the menu.
