# Page Inventory & Migration Tracker

**Source:** [Migration tracker (Google Sheet)](https://docs.google.com/spreadsheets/d/13jFsEzuXJp5MbTvwvQ1JmuTFIeLlGJ3zLXCSAUlMf4A/edit?usp=sharing)  
**Imported from:** `goconstellation.com Migration Tracker - Page Types.csv` (Page Types tab)  
**Last updated:** 2026-05-18

Use this doc for **template definitions**, **standalone page mapping**, and **category scale**. Per-URL status (done/remaining) lives on the spreadsheet’s other tabs—export those tabs into this repo when you reconcile scope.

**Related:** [TODO.md](./TODO.md) · [PROGRESS.md](./PROGRESS.md) · [TEMPLATES.md](./TEMPLATES.md)

---

## Tracker scope (all tabs)

| Tab (sheet) | Purpose | In this repo |
|-------------|---------|--------------|
| **Page Types** | Templates + standalone pages | Documented below (from CSV export) |
| **Blog Posts** | Per-post migration status | Reconcile against `src/pages/blog/` (~160 files) |
| **City / Location** | Per-city URLs | Reconcile against ~121 city `.astro` files |
| **Service Pages** | Service line URLs | ~9 of ~18 built in repo—**export tab for exact URL list** |
| **Case Studies** | Narrative case study URLs | ~27 in `src/pages/case-study/` |
| **Podcast Episodes** | Episode URLs | ~23 in `src/pages/podcasts/` |
| **Practice Areas** | Practice-area URLs | ~12 using `PracticeAreaTemplate` |
| **Core / Generic** | Utility & hub pages | See [Standalone pages](#standalone-pages) |

**Manifest total (sheet):** **325** URLs tracked · **172** marked done · **153** remaining ([linear-progress.md](./linear-progress.md))

---

## Templates (from Page Types tab)

Each template is a full-page shell: `BaseLayout` → `SiteHeader` → content → `SiteFooter`.

| Template file | Page type | Est. pages | Layout pattern | Example URL |
|---------------|-----------|------------|----------------|-------------|
| `BlogPostTemplate.astro` | Blog posts | **132** (sheet) / **160+** (repo) | Hero + 2-col article + sidebar (CTA + chapter TOC) | `/blog/law-firm-seo-guide/` |
| `BookACallTemplate.astro` | Book a call / landing | 1 | Focused CTA layout | `/book-call/` (see note below) |
| `CaseStudyNarrativeTemplate.astro` | Case studies | **29** (sheet) / **27** (repo) | Hero + narrative results story | `/seo-growth-case-study-sabbeth-law/` |
| `CityTemplate.astro` | City / location SEO | **118** | Localized hero + service content | `/law-firm-marketing-atlanta-ga/` |
| `CoreTemplate.astro` | Core / general | **6+** hubs | Hero + credibility badge bar + slot | `/about-us/`, `/blog/`, `/podcasts/` |
| `HomepageTemplate.astro` | Homepage | 1 (+ v2, v3 variants) | Full homepage sections | `/` |
| `PodcastTemplate.astro` | Podcast episodes | **22** | Episode hero + embed + platform CTAs | `/podcasts/law-firm-seo-2026/` |
| `PracticeAreaTemplate.astro` | Practice areas | **9** (sheet) / **12** (repo) | Practice hero + long-form content | `/personal-injury-law-firm-marketing/` |
| `ServiceTemplate.astro` | Service lines | **~18** (manifest) / **6** (repo) | Service hero + features + CTAs | `/law-firm-seo/`, `/law-firm-ppc/` |

**Also in repo (not on Page Types tab):**

| Template | Purpose | Est. usage |
|----------|---------|------------|
| `GuidesTemplate.astro` | Pillar guides with chapter nav | Few (`law-firm-seo/*`, etc.) |
| `AgencyReviewTemplate.astro` | Competitor / agency reviews | ~11 under `/blog/` |
| `HomepageTemplate-v2.astro` / `v3` | Migration / Backstop experiments | `/examples/` only |

### Template layout patterns (detail)

```text
BlogPostTemplate
  BaseLayout → SiteHeader → hero → 2-col (article + sidebar CTA/TOC) → bottom CTA → SiteFooter

CityTemplate
  BaseLayout → SiteHeader → localized hero → service content → SiteFooter

ServiceTemplate
  BaseLayout → SiteHeader → service hero → features → CTA sections → SiteFooter

PodcastTemplate
  BaseLayout → SiteHeader → episode hero → embed → show notes → platform links → SiteFooter

CoreTemplate
  BaseLayout → SiteHeader → hero → badge bar → slot → SiteFooter

CaseStudyNarrativeTemplate
  BaseLayout → SiteHeader → hero → narrative content → SiteFooter
```

### Book-a-call note

The tracker lists `BookACallTemplate` for `/book-call/`. In the repo, **production** `src/pages/book-call.astro` is a **custom** long-form page with LeadConnector iframe—not the simpler `/examples/book-a-call-example/` route. Treat book-call as **built** but **needs content/QA**, not template migration.

---

## Standalone pages

Explicit page files from the Page Types tab (not bulk-generated routes).

| Page file | URL | Tracker layout | Post-migration focus |
|-----------|-----|----------------|-------------------|
| `src/pages/index.astro` | `/` | `HomepageTemplate` | Backstop parity, `data-section`, confirm v1 not v2/v3 |
| `src/pages/about-us.astro` | `/about-us/` | `CoreTemplate` | Fix `title` prop bug; content QA |
| `src/pages/blog/index.astro` | `/blog/` | `CoreTemplate` | Fix `title` bug; sync manual post list |
| `src/pages/podcasts/index.astro` | `/podcasts/` | `CoreTemplate` | Fix `title` bug; grid links to all episodes |
| `src/pages/case-studies.astro` | `/case-studies/` | **BaseLayout (custom)** | Hub IA + links to 27 case studies |
| `src/pages/book-call.astro` | `/book-call/` | BookACallTemplate* | GHL embed, WP testimonial images, mobile |
| `src/pages/services.astro` | `/services/` | **BaseLayout (custom)** | Service hub links vs manifest (~9 missing) |
| `src/pages/law-firm-seo-services.astro` | `/law-firm-seo-services/` | **BaseLayout (custom)** | Flagship visual + meta QA |
| `src/pages/law-firm-ppc.astro` | `/law-firm-ppc/` | **BaseLayout (custom)** | Flagship visual + meta QA |
| `src/pages/law-firm-web-design.astro` | `/law-firm-web-design/` | **BaseLayout (custom)** | Flagship visual + meta QA |
| `src/pages/faq.astro` | `/faq/` | `CoreTemplate` | Fix `title` bug; accordion/content QA |

**Additional standalone pages (repo; confirm on manifest tabs):**

| Page file | URL | Notes |
|-----------|-----|--------|
| `privacy-policy.astro` | `/privacy-policy/` | `CoreTemplate` · title bug |
| `terms-of-service.astro` | `/terms-of-service/` | `CoreTemplate` · title bug |
| `support-center.astro` | `/support-center/` | `CoreTemplate` · ClickUp iframe · title bug |
| `thank-you.astro` | `/thank-you/` | Post-booking · GHL redirect QA |
| `become-a-partner.astro` | `/become-a-partner/` | Partner flow QA |
| `partner-program.astro` | `/partner-program/` | Partner flow QA |
| `referral-program.astro` | `/referral-program/` | Referral flow QA |
| `patrick-carver.astro` | `/patrick-carver/` | Bio page QA |
| `lead-flow-mastery-guide.astro` | `/lead-flow-mastery-guide/` | Guide QA |

---

## Service pages (built vs manifest)

**Built in repo today (9):**

| Page | Implementation | Priority |
|------|----------------|----------|
| `/law-firm-seo-services/` | Custom flagship | P0 edit + QA |
| `/law-firm-ppc/` | Custom flagship | P0 edit + QA |
| `/law-firm-web-design/` | Custom flagship | P0 edit + QA |
| `/meta-ads/` | `ServiceTemplate` | P0 build QA |
| `/law-firm-branding/` | `ServiceTemplate` | P1 |
| `/email-marketing-for-law-firms/` | `ServiceTemplate` | P1 |
| `/social-media-for-law-firms/` | `ServiceTemplate` | P1 |
| `/ai-chatbot-for-law-firms/` | `ServiceTemplate` | P1 |
| `/bilingual-seo-law-firms/` | `ServiceTemplate` | P1 |

**Manifest gap:** Tracker shows **Service Pages 1 / 4** on one view and **~18** service URLs in scope elsewhere—**~9 URLs likely still missing** in repo. Export the **Service Pages** sheet tab and add rows to [TODO.md](./TODO.md) § Missing service URLs.

---

## Category progress (sheet vs repo)

| Category | Sheet (done/total) | Repo reality | Action |
|----------|-------------------|--------------|--------|
| Homepage | 1/1 | 1 | Visual QA only |
| Podcast episodes | 22/22 | ~23 | Fix empty embeds; QA |
| Case studies | 26/29 | ~27 | 2–3 sheet rows vs repo mismatch |
| Core pages | 5/6 | 6+ | Fix `CoreTemplate` titles |
| Blog posts | 107/132 | ~160 files | Reconcile sheet; sampled QA |
| Practice areas | 6/9 | ~12 | Mark complete or build 0–3 |
| Service pages | 1/4* | 9 built | Export tab; build missing |
| City / location | 4/118 | ~121 files | Sheet stale—mark migrated; sample QA |
| Generic / other | 0/4 | ~13 files | Mark complete on sheet |

\*Sheet category counts may not match full manifest service list—use all tabs for authority.

---

## Nested routes (repo; include in QA samples)

| Path prefix | ~Count | Template mix |
|-------------|--------|----------------|
| `src/pages/law-firm-seo/*` | 13 | `BlogPostTemplate`, `GuidesTemplate` |
| `src/pages/lawyer-advertising/*` | 3 | Article-shaped |
| `src/pages/law-firm-web-design/*` | 1 | Tie to web-design service QA |
| `src/pages/examples/*` | 10 | Dev/QA only—**noindex before launch** |

---

## How to refresh this inventory

1. Export each Google Sheet tab as CSV into `reference/` (optional) or update tables here.
2. Re-run repo counts:

```bash
find src/pages -name '*.astro' | wc -l
find src/pages/blog -maxdepth 1 -name '*.astro' ! -name 'index.astro' | wc -l
find src/pages -maxdepth 1 -name 'attorney-marketing-*.astro' | wc -l
find src/pages -maxdepth 1 -name 'law-firm-marketing-*.astro' | wc -l
```

3. Update [PROGRESS.md](./PROGRESS.md), [linear-progress.md](./linear-progress.md), and [TODO.md](./TODO.md).
