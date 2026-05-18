# Templates Guide

This guide explains every page layout in `src/layouts/templates/`: what each one is for, which props it accepts, how content gets into it, and how to pick the right template for a new page.

For local setup and editing workflows, see [DEVELOPMENT.md](./DEVELOPMENT.md). For the big picture, see [SITE-OVERVIEW.md](./SITE-OVERVIEW.md). For step-by-step content authoring, see [CONTENT-GUIDE.md](./CONTENT-GUIDE.md).

---

## How templates fit together

Every template is a **full-page shell**. None of them are small widgets. You import one at the top of a page file, pass configuration as props, and optionally pass HTML as **slot content** (the markup between opening and closing template tags).

All templates eventually render through this stack:

```
Your page file (src/pages/...)
        ↓
A template (src/layouts/templates/...)
        ↓
BaseLayout  →  <head>, SEO meta, global CSS
SiteHeader  →  green pill navigation
[your content]
SiteFooter  →  footer links
```

**`BaseLayout`** (`src/layouts/BaseLayout.astro`) is not a page template. It is the HTML document wrapper. Templates import it and pass SEO fields. Its main prop is `title` (the browser tab title and OG title). Most templates map their `seoTitle` prop to `title={seoTitle}` on `BaseLayout`.

**`SiteHeader`** and **`SiteFooter`** are shared across templates (except where a page builds everything manually. See [Pages without templates](#pages-without-templates) below).

---

## Quick picker: which template do I need?

| You are building… | Use this template |
|-----------------|-------------------|
| Homepage | `HomepageTemplate` (via `src/pages/index.astro`) |
| Blog article | `BlogPostTemplate` |
| Long guide under a practice area (PI, criminal defense, immigration…) | `PracticeAreaTemplate` |
| City / local landing page (`attorney-marketing-denver`, etc.) | `CityTemplate` |
| Service marketing page (SEO, PPC, branding) with structured sections | `ServiceTemplate` |
| Competitor / agency review post | `AgencyReviewTemplate` |
| Pillar guide with chapter nav (e.g. law firm marketing guide) | `GuidesTemplate` |
| Podcast episode | `PodcastTemplate` |
| Client case study narrative | `CaseStudyNarrativeTemplate` |
| Book-a-call landing (template version) | `BookACallTemplate` |
| Simple marketing page (about, FAQ, privacy, blog index) | `CoreTemplate` |
| Huge custom page not matching anything above | `BaseLayout` + header/footer directly |

**Live references:** Every major template has an example route under `/examples/` (e.g. `/examples/blog-post-example/`). Use those when you want to see a working page without hunting through production URLs.

---

## Shared SEO props

Nearly every template accepts some combination of these:

| Prop | Required? | Purpose |
|------|-----------|---------|
| `seoTitle` | Yes | Page `<title>` and social titles. Often ends with `\| Constellation Marketing` |
| `seoDescription` | No | Meta description |
| `canonicalUrl` | No | Canonical link; should match production URL when known |
| `ogImage` | No | Social preview image; defaults in `BaseLayout` to `/images/constellation-og.jpg` |

**Convention:** Set `canonicalUrl` to the final production URL (`https://goconstellation.com/...`) even when testing on staging, unless you intentionally want otherwise.

---

## Import paths (easy to get wrong)

The depth of `../` depends on where your page file lives:

| Page location | Import path |
|---------------|-------------|
| `src/pages/foo.astro` | `../layouts/templates/SomeTemplate.astro` |
| `src/pages/blog/foo.astro` | `../../layouts/templates/SomeTemplate.astro` |
| `src/pages/case-study/foo.astro` | `../../layouts/templates/SomeTemplate.astro` |
| `src/pages/examples/foo.astro` | `../../layouts/templates/SomeTemplate.astro` |

---

## Template reference

### `HomepageTemplate` (and v2, v3)

**File:** `src/layouts/templates/HomepageTemplate.astro`  
**Example:** `/examples/homepage-example/`  
**Used by:** `src/pages/index.astro` (live homepage)

**Props:**

| Prop | Required? | Default | Notes |
|------|-----------|---------|-------|
| `seoTitle` | Yes | (none) | |
| `seoDescription` | No | `''` | |
| `canonicalUrl` | No | (none) | |

**Content model:** There is **no slot**. The entire homepage (hero, testimonials, FAQ, services grid, etc.) lives inside the template file (~1,500 lines). To change homepage copy or layout, edit the template directly (or the v2/v3 variant if you are running a migration experiment).

**Homepage v2 / v3:** `HomepageTemplate-v2.astro` and `HomepageTemplate-v3.astro` exist for migration and Backstop comparison work. Only `index.astro` matters for production; confirm which template it imports before editing.

```astro
---
import HomepageTemplate from '../layouts/templates/HomepageTemplate.astro';
---

<HomepageTemplate
  seoTitle="A Law Firm Marketing Agency That Actually Pays for Itself | Constellation Marketing"
  seoDescription="More Revenue. More High-Value Cases. Trusted by 85+ Law Firms Nationwide."
  canonicalUrl="https://goconstellation.com/"
/>
```

---

### `BlogPostTemplate`

**File:** `src/layouts/templates/BlogPostTemplate.astro`  
**Example:** `/examples/blog-post-example/`  
**Used by:** Most files in `src/pages/blog/`, plus some nested guides (`law-firm-seo/*`, `lawyer-advertising/*`) that are article-shaped.

**Props:**

| Prop | Required? | Default | Notes |
|------|-----------|---------|-------|
| `seoTitle` | Yes | (none) | |
| `seoDescription` | No | `''` | |
| `canonicalUrl` | No | (none) | |
| `ogImage` | No | (none) | |
| `pageTitle` | No | First segment of `seoTitle` before `\|` | Hero H1 |
| `publishDate` | No | (none) | ISO date string, e.g. `2025-12-07` |
| `author` | No | `Patrick Carver` | |
| `authorTitle` | No | `CEO & Founder` | |
| `authorImage` | No | Patrick headshot path | |
| `category` | No | `Law Firm Marketing` | |
| `featuredImage` | No | (none) | Hero / card image |
| `readTime` | No | `8 min read` | |
| `chapters` | No | `[]` | Table of contents: `{ label, anchor }[]` |

**Slots:**

| Slot | Purpose |
|------|---------|
| Default slot | Full article body as HTML (`<h2>`, `<p>`, `<ul>`, images, etc.) |

**Features built into the template:** Author byline, optional featured image, chapter sidebar (when `chapters` is set), article typography (`.article-body`), reading-friendly gray background.

**Chapter anchors:** Each `chapters` entry needs a matching `id` on a heading in the body, e.g. `{ label: "SEO Basics", anchor: "seo-basics" }` → `<h2 id="seo-basics">`.

```astro
<BlogPostTemplate
  seoTitle="Winning Law Firm SEO: The Definitive Guide"
  canonicalUrl="https://goconstellation.com/law-firm-seo/"
  publishDate="2025-12-07"
  chapters={[
    { label: "What Is Law Firm SEO?", anchor: "what-is-law-firm-seo" },
  ]}
>
  <h2 id="what-is-law-firm-seo">What Is Law Firm SEO?</h2>
  <p>Body copy here…</p>
</BlogPostTemplate>
```

---

### `PracticeAreaTemplate`

**File:** `src/layouts/templates/PracticeAreaTemplate.astro`  
**Example:** `/examples/practice-area-example/`  
**Used by:** Top-level practice pages, such as `criminal-defense-marketing.astro`, `immigration-law-firm-marketing.astro`, `personal-injury-lawyer-marketing.astro`, etc.

**Props:**

| Prop | Required? | Default | Notes |
|------|-----------|---------|-------|
| `seoTitle` | Yes | (none) | |
| `seoDescription` | No | `''` | |
| `canonicalUrl` | No | (none) | |
| `ogImage` | No | (none) | |
| `pageTitle` | No | From `seoTitle` | Hero headline |
| `practiceArea` | No | `Law Firm Marketing` | Badge label in hero |
| `heroSubtitle` | No | Generic Constellation line | |
| `publishDate` | No | (none) | |
| `readTime` | No | `10 min read` | |
| `chapters` | No | `[]` | Same shape as blog |
| `caseStudies` | No | `[]` | Sidebar / proof cards (see below) |

**`caseStudies` object shape:**

```ts
{
  client: string;
  keyResult: string;
  practiceArea: string;
  service: string;
  teaser: string;
  afterPoints: string[];
  url: string;  // link to /case-study/...
}
```

**Slots:**

| Slot | Purpose |
|------|---------|
| Default slot | Long-form guide HTML |

**Features:** Dark navy hero, green reading progress bar at top of viewport, chapter nav, optional case study cards tied to the practice area. Supports special classes in content such as `stat-callout`, `cta-inset` (see practice-area example page).

---

### `CityTemplate`

**File:** `src/layouts/templates/CityTemplate.astro`  
**Example:** `/examples/city-example/`  
**Used by:** `attorney-marketing-*` and `law-firm-marketing-*` pages (~130+ URLs).

**Props:**

| Prop | Required? | Default | Notes |
|------|-----------|---------|-------|
| `seoTitle` | Yes | (none) | |
| `seoDescription` | No | `''` | |
| `canonicalUrl` | No | (none) | |
| `ogImage` | No | (none) | |
| `city` | No | `Your City` | Injected into hero H1 |
| `state` | No | `''` | |
| `heroImage` | No | Albuquerque default | Background image for hero |
| `heroSubtitle` | No | Generic line | Under the H1 |
| `landscapeImage` | No | Albuquerque default | Secondary scenic image in layout |

**Slots:** None; the page body is entirely in the template.

**Important quirk:** Many city page files pass `serviceImages={[...]}` because `scripts/build_city_pages.py` generates that prop. The current `CityTemplate` **does not declare or use** `serviceImages`; those values are ignored. To change city imagery, edit `heroImage`, `landscapeImage`, or the template itself until the prop is wired up.

```astro
<CityTemplate
  seoTitle="Attorney Marketing in Austin | Constellation Marketing"
  city="Austin"
  state="Texas"
  heroImage="/images/attorney-marketing-austin-title.webp"
  landscapeImage="/images/austin-building.webp"
  canonicalUrl="https://goconstellation.com/attorney-marketing-austin/"
/>
```

---

### `ServiceTemplate`

**File:** `src/layouts/templates/ServiceTemplate.astro`  
**Example:** `/examples/service-example/`  
**Used by:** `meta-ads.astro`, `law-firm-branding.astro`, `email-marketing-for-law-firms.astro`, `social-media-for-law-firms.astro`, `ai-chatbot-for-law-firms.astro`, and similar service-line pages.

**Props:** All SEO props above, plus many **optional section props with defaults**, so a minimal page can pass only `seoTitle`, `seoDescription`, and `canonicalUrl` and get a complete layout.

| Prop | Purpose |
|------|---------|
| `heroTagline` | Small line above title |
| `pageTitle` | Main H1 |
| `heroSubtitle` | Under title |
| `heroImage`, `heroImageAlt`, `heroImageMaxWidth`, `heroImageAlign` | Hero visual |
| `growthTitle`, `growthIntro`, `growthBullets` | “How our services fuel growth” block |
| `drivesCasesTitle`, `drivesCasesDesc`, `drivesCasesBullets` | Cases / outcomes section |
| `strategyTitle`, `strategyCards` | Card grid (`{ title, desc }[]`) |
| `ctaBannerTitle` | Mid-page CTA strip |
| `faqs` | `{ q, a }[]` accordion content |

**Slots:** None; customize via props or edit the template for one-off service pages.

**Note:** Major service URLs like `/law-firm-seo-services/` and `/law-firm-ppc/` still use **hand-built** `BaseLayout` pages (not this template). Check the target URL before assuming `ServiceTemplate`.

---

### `AgencyReviewTemplate`

**File:** `src/layouts/templates/AgencyReviewTemplate.astro`  
**Example:** No dedicated `/examples/` file; see `src/pages/blog/rankings-io-review.astro`  
**Used by:** Competitor reviews, such as `rankings-io-review`, `paperstreet-review`, `juris-digital-review`, `webris-review`, `smb-team-review`, `on-the-map-review`, etc.

**Props:**

| Prop | Purpose |
|------|---------|
| `pageTitle`, `pageSubtitle` | Hero |
| `agencyName`, `agencyLocation`, `agencyFounded`, `agencySpecialty`, `agencyDescription` | Who is being reviewed |
| `googleRating`, `googleReviewCount` | Review summary |
| `positiveThemes`, `negativeThemes` | Bullet lists |
| `positiveSummary`, `negativeSummary` | Prose blocks |
| `reviewSource`, `reviewDate` | Attribution |
| `comparisonRows` | `{ feature, ours, theirs, note? }[]` comparison table |
| `extraChapters` | Extra TOC entries for slot headings |
| `ctaHeadline`, `ctaSubtext` | Bottom CTA |

**Slots:**

| Slot | Purpose |
|------|---------|
| Default slot | Additional review sections (pricing, verdict, etc.) |

The template builds much of the page from props: agency overview, Google review themes, comparison table, and similar blocks. Use the slot for content that does not fit the structured fields.

---

### `GuidesTemplate`

**File:** `src/layouts/templates/GuidesTemplate.astro`  
**Example:** See `src/pages/blog/law-firm-marketing-guide.astro` or `law-firm-advertising-guide.astro`  
**Used by:** Large pillar guides with a fixed chapter list.

**Props:**

| Prop | Required? | Default | Notes |
|------|-----------|---------|-------|
| `seoTitle` | Yes | (none) | |
| `guideTitle` | Yes | (none) | Display title |
| `chapters` | Yes | `[]` | `{ id, label }[]` (use `id`, not `anchor`) |
| `guideSubtitle` | No | `''` | |
| `heroImage`, `heroImageAlt` | No | (none) | |
| `publishDate` | No | (none) | |
| `authorName`, `authorImage` | No | Patrick defaults | |

**Slots:**

| Slot | Purpose |
|------|---------|
| Default slot | Guide body; use `id` on headings matching `chapters[].id` |

Similar to `PracticeAreaTemplate` in feel but focused on **editorial guides** rather than a single practice area landing page.

---

### `PodcastTemplate`

**File:** `src/layouts/templates/PodcastTemplate.astro`  
**Example:** `/examples/podcast-example/`  
**Used by:** `src/pages/podcasts/*.astro` (and occasionally a blog post that reuses the layout).

**Props:**

| Prop | Purpose |
|------|---------|
| `pageTitle` | Episode title |
| `publishDate` | |
| `duration` | Shown in meta row |
| `embedUrl` | iframe `src` for the player |
| `spotifyUrl`, `youtubeUrl`, `appleUrl`, `audibleUrl` | Platform links |
| `shortVideos` | `{ title, url }[]` optional clip list |

**Slots:**

| Slot | Purpose |
|------|---------|
| Default slot | Show notes (HTML) |

**Features:** Player iframe, platform buttons, styled show-notes area (`.show-notes` typography).

---

### `CaseStudyNarrativeTemplate`

**File:** `src/layouts/templates/CaseStudyNarrativeTemplate.astro`  
**Used by:** `src/pages/case-study/*.astro`

**Props:**

| Prop | Required? | Notes |
|------|-----------|-------|
| `seoTitle` | Yes | |
| `clientName` | Yes | |
| `pageTitle` | Yes | Story headline |
| `keyResult` | Yes | Highlight outcome |
| `clientUrl` | No | External firm URL |
| `heroImage` | No | |
| `publishDate` | No | |
| `practiceArea` | No | |
| `services` | No | string[] tags |
| `beforePoints`, `afterPoints` | No | Bullet lists |
| `clientQuote` | No | Pull quote |
| `relatedCases` | No | `{ title, url, practiceArea? }[]` |

**Slots:**

| Slot | Purpose |
|------|---------|
| `before` | Narrative before results |
| `after` | Narrative after results |

Use named slots for the story; use props for structured metadata and sidebar-style facts.

---

### `BookACallTemplate`

**File:** `src/layouts/templates/BookACallTemplate.astro`  
**Example:** `/examples/book-a-call-example/`  
**Used by:** Example route; production `/book-call/` uses a **custom** page instead (see below).

**Props:**

| Prop | Default | Notes |
|------|---------|-------|
| `pageTitle` | Book Your Free Strategy Call | |
| `subtitle` | Default pitch line | |
| `calendarUrl` | (none) | Booking iframe URL when set |

Includes trust bullets and testimonials baked into the template. If you change the live booking widget, check both this template and `src/pages/book-call.astro`.

---

### `CoreTemplate`

**File:** `src/layouts/templates/CoreTemplate.astro`  
**Example:** `/examples/core-example/`  
**Used by:** `about-us.astro`, `faq.astro`, `privacy-policy.astro`, `terms-of-service.astro`, `support-center.astro`, `podcasts/index.astro`, `blog/index.astro`, and other “simple hero + body” pages.

**Props:**

| Prop | Required? | Default | Notes |
|------|-----------|---------|-------|
| `seoTitle` | Yes | (none) | |
| `heroTitle` | Yes | (none) | Big green headline |
| `heroSuperTitle` | No | Trusted by 85+… | |
| `heroSubtitle` | No | Generic marketing line | |
| `heroCta` | No | Get Started Now | |
| `heroCtaHref` | No | `/book-call/` | |
| `heroImage` | No | Default composite | Used if `hero-image` slot empty |
| `hideHeroImage` | No | `false` | |

**Slots:**

| Slot | Purpose |
|------|---------|
| `hero-extra` | Stats row, badges, etc. above CTA |
| `hero-image` | Replace right-column image |
| Default slot | Everything below hero (and below badge row) |

**Features:** Hero with trust badges, then your custom content. `blog/index.astro` demonstrates named slots for a featured post grid.

**Note:** `CoreTemplate` passes props to `BaseLayout`; when adding new pages, ensure `title={seoTitle}` is used on `BaseLayout` (the template should handle this; if the tab title is blank, check here).

---

## Pages without templates

Some high-traffic pages are **custom-built** with `BaseLayout` + `SiteHeader` + `SiteFooter` only:

| Page | Why custom |
|------|------------|
| `book-call.astro` | Lead Connector booking iframe and bespoke layout |
| `case-studies.astro` | Large filterable/grid index |
| `law-firm-seo-services.astro` | Very long, service-specific sections |
| `law-firm-ppc.astro` | Same |
| `law-firm-web-design.astro` | Same |
| `become-a-partner.astro`, `thank-you.astro` | One-offs |

For these, copy an existing custom page as a starting point rather than forcing a template.

---

## Slots cheat sheet

| Template | Default slot | Named slots |
|----------|--------------|-------------|
| `BlogPostTemplate` | Article HTML | (none) |
| `PracticeAreaTemplate` | Guide HTML | (none) |
| `GuidesTemplate` | Guide HTML | (none) |
| `PodcastTemplate` | Show notes | (none) |
| `AgencyReviewTemplate` | Extra sections | (none) |
| `CaseStudyNarrativeTemplate` | (none) | `before`, `after` |
| `CoreTemplate` | Main body | `hero-extra`, `hero-image` |
| `CityTemplate`, `ServiceTemplate`, `HomepageTemplate*` | (none) | ,  |

In Astro, **default slot** = content between tags with no attribute. **Named slot** = child element with `slot="hero-extra"` in the page, `<slot name="hero-extra" />` in the template.

---

## `/examples/` routes

These exist for development and visual QA, not for public navigation:

| Route | Template |
|-------|----------|
| `/examples/homepage-example/` | `HomepageTemplate` |
| `/examples/homepage-v2-example/` | `HomepageTemplate-v2` |
| `/examples/homepage-v3-example/` | `HomepageTemplate-v3` |
| `/examples/blog-post-example/` | `BlogPostTemplate` |
| `/examples/practice-area-example/` | `PracticeAreaTemplate` |
| `/examples/city-example/` | `CityTemplate` |
| `/examples/service-example/` | `ServiceTemplate` |
| `/examples/core-example/` | `CoreTemplate` |
| `/examples/book-a-call-example/` | `BookACallTemplate` |
| `/examples/podcast-example/` | `PodcastTemplate` |

BackstopJS compares production to `/examples/homepage-example/` by default ([DEVELOPMENT.md](./DEVELOPMENT.md#optional-visual-regression-backstopjs)).

---

## Common mistakes

1. **Wrong import depth**: `../../` vs `../` breaks the build with “Cannot find module”.
2. **Mismatched chapter IDs**: `chapters[].anchor` (blog/practice) or `chapters[].id` (guides) must match heading `id` attributes exactly.
3. **Assuming `serviceImages` works on city pages**: prop is currently ignored; update `heroImage` instead.
4. **Using `BlogPostTemplate` for a page that needs comparison tables or agency metadata**: use `AgencyReviewTemplate`.
5. **Editing only the thin page file on homepage or city pages**: most markup is in the template; the page file may only pass SEO props.
6. **Linking `/examples/` from production nav**: don’t.

---

## Adding a new page (checklist)

1. Decide template from [Quick picker](#quick-picker-which-template-do-i-need) above.
2. Create `src/pages/your-slug.astro` (or nested path).
3. Copy props from the matching `/examples/` page or a similar production page.
4. Set `canonicalUrl` to the final production URL.
5. Add body content in the slot (if the template has one).
6. Run `npm run dev` and open the URL.
7. Run `npm run build` before merging.

For bulk city or blog pages, see migration scripts in [DEVELOPMENT.md](./DEVELOPMENT.md#optional-python-migration-scripts).

---

## Related documentation

| Document | Topics |
|----------|--------|
| [DEVELOPMENT.md](./DEVELOPMENT.md) | Commands, troubleshooting, Backstop |
| [SITE-OVERVIEW.md](./SITE-OVERVIEW.md) | Architecture and stack |
| [REFORE-APPROACH.md](../REFORE-APPROACH.md) | Homepage migration from WordPress |
