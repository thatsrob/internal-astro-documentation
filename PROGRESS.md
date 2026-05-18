# Migration Progress

**Last audited:** 2026-05-18 (against the `main` branch in this repo)  
**Purpose:** Compare the team’s migration targets to what is actually in the codebase, and list what is left before production cutover.

| Resource | Link |
|----------|------|
| **Migration tracker (source of targets)** | [Google Sheet](https://docs.google.com/spreadsheets/d/13jFsEzuXJp5MbTvwvQ1JmuTFIeLlGJ3zLXCSAUlMf4A/edit?usp=sharing) |
| **Page inventory (templates + standalone)** | [PAGE-INVENTORY.md](./PAGE-INVENTORY.md) |
| **Post-migration to-do** | [TODO.md](./TODO.md) |
| **Staging** | [staging.goconstellation.com](https://staging.goconstellation.com) |
| **Repo** | [github.com/ConstellationMarketing/con-staging](https://github.com/ConstellationMarketing/con-staging) |
| **Production (live today)** | [goconstellation.com](https://www.goconstellation.com) — WordPress/Divi |

---

## Executive summary

| Lens | Estimate | Notes |
|------|----------|--------|
| **Tracker (team spreadsheet)** | **~75% complete** | Phase 3 still lists large content backlogs (blog, city, etc.). |
| **Repo (files in `src/pages/`)** | **~88% of manifest routes exist as `.astro` pages** | Many categories appear **more complete in Git than the tracker reflects**. |
| **Launch-ready** | **~55–65%** | Most HTML exists; **QA, SEO/analytics, CSS weight, and cutover** work remain substantial. |

**Bottom line:** Content migration in the repo is **ahead of the ~75% tracker status** for blogs, cities, case studies, and podcasts. The gap to launch is less “write 300 pages from scratch” and more **reconcile the spreadsheet, finish service/flagship pages, polish/QA, and complete Phase 4–5**.

---

## Migration overview

Migrating **goconstellation.com** from WordPress/Divi to **Astro (SSG) + Cloudflare Pages**.

| Item | Target |
|------|--------|
| **Manifest page count (tracker)** | **325** URLs |
| **`.astro` routes in repo** | **396** files under `src/pages/` (**386** excluding `/examples/*`) |
| **Deploy target** | Cloudflare Pages project `con-staging` |
| **Primary conversion** | `/book-call/` (LeadConnector / GoHighLevel) |

The repo route count is **higher than 325** because it includes hub/nested sections (e.g. `law-firm-seo/*`, `lawyer-advertising/*`), template example routes, and pages that may not be rows in the spreadsheet. Treat the **tracker as the scope authority**; use the **repo audit** to see what is already built.

---

## Phase status

### Phase 1 — Scrape & inventory — **Done**

- Full site manifest in the migration tracker (325 pages tagged by template, cluster, tier, keyword).
- Repo structure matches a file-per-URL Astro model.

**Nothing blocking** on this phase.

---

### Phase 2 — Templates — **Done** (with extra templates in repo)

**Tracker:** 7 templates built and pixel-polished — homepage, service, practice-area, blog-post, podcast, city, book-a-call.

**Repo (`src/layouts/templates/`):** **13** template files exist:

| Template | Tracker “core 7” | In repo |
|----------|------------------|---------|
| `HomepageTemplate` | Yes | Yes (+ v2, v3 for migration/Backstop) |
| `ServiceTemplate` | Yes | Yes |
| `PracticeAreaTemplate` | Yes | Yes |
| `BlogPostTemplate` | Yes | Yes |
| `PodcastTemplate` | Yes | Yes |
| `CityTemplate` | Yes | Yes |
| `BookACallTemplate` | Yes | Yes |
| `CoreTemplate` | — | Yes (about, FAQ, privacy, blog index, etc.) |
| `CaseStudyNarrativeTemplate` | — | Yes |
| `GuidesTemplate` | — | Yes |
| `AgencyReviewTemplate` | — | Yes |

**Still to do (template layer):**

- [ ] Fix `CoreTemplate` → `BaseLayout` prop bug (`seoTitle` vs `title`) on several live pages ([LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md)).
- [ ] Confirm production homepage uses intended template (`index.astro` → `HomepageTemplate`, not v2/v3).
- [ ] Decide fate of `HomepageTemplate-v2` / `v3` (Backstop only vs delete after cutover).

---

### Phase 3 — Core pages on staging — **Mostly done in repo; tracker understates progress**

#### Repo audit (2026-05-18)

| Category | Tracker said (remaining) | In repo now | Assessment |
|----------|--------------------------|-------------|------------|
| **Blog posts** | 115 remaining | **160** files in `src/pages/blog/` (excl. index) | **Likely ahead of tracker** — reconcile sheet |
| **City pages** | 117 remaining | **121** (`64` `attorney-marketing-*` + `57` `law-firm-marketing-*`) | **~complete in repo** |
| **Case studies** | 21 remaining | **27** in `src/pages/case-study/` | **Complete or over-built** vs tracker |
| **Podcast episodes** | 21 remaining | **23** in `src/pages/podcasts/` (excl. index) | **~complete in repo** |
| **Practice areas** | 6 complete (tracker) | **12** using `PracticeAreaTemplate` | **Ahead of “6”** |
| **Service pages** | ~15 remaining (3 of ~18 done) | **3** flagship custom + **6** `ServiceTemplate` + overlaps | **Still the main content gap** |
| **Generic / utility** | 12 remaining (tracker) | **13** standalone pages (about, FAQ, legal, support, etc.) | **Mostly present** |

**Blog breakdown in repo:**

| Template | Count |
|----------|------:|
| `BlogPostTemplate` | 129 |
| `AgencyReviewTemplate` | 11 |
| `GuidesTemplate` | 3 |
| `PodcastTemplate` (blog path) | ~10 |
| Other / mixed | ~17 |
| `blog/index` | 1 |

**Service pages in repo:**

| Page | Implementation |
|------|----------------|
| `law-firm-seo-services.astro` | Custom (`BaseLayout` + inline — flagship) |
| `law-firm-ppc.astro` | Custom (`BaseLayout` + inline — flagship) |
| `law-firm-web-design.astro` | Custom (`BaseLayout` + inline — flagship) |
| `meta-ads`, `law-firm-branding`, `email-marketing-for-law-firms`, `social-media-for-law-firms`, `ai-chatbot-for-law-firms`, `bilingual-seo-law-firms` | `ServiceTemplate` |

**Tracker vs repo action:** Update the Google Sheet so remaining counts match Git (especially blog and city). Otherwise progress reporting will stay stuck at ~75% while the repo is further along.

#### Phase 3 — Still to do (content & parity)

- [ ] **Reconcile tracker** with repo (325 manifest rows ↔ 386 non-example routes).
- [ ] **Service line pages** — confirm which of ~18 manifest URLs lack a page or still match WordPress only.
- [ ] **Blog** — if manifest lists posts not in repo, run `scripts/build_blog_posts.py` / `scripts/build_astro_post.py`; if repo has posts not in manifest, mark complete or remove duplicates.
- [ ] **City** — spot-check a sample against production (canonical URLs, hero images, `CityTemplate` props).
- [ ] **Case studies / podcasts** — content QA (some podcasts have **empty `embedUrl`** e.g. `podcasts/how-to-close-more-cases.astro`).
- [ ] **Generic pages** — verify `services.astro`, partner/referral flows, `patrick-carver.astro`, `lead-flow-mastery-guide.astro` against manifest.
- [ ] **Images** — many pages still hotlink `goconstellation.com/wp-content/uploads/`; migrate critical paths before WordPress shutdown ([INTEGRATIONS.md](./INTEGRATIONS.md)).
- [ ] **`/examples/*`** — 10 routes ship in production builds; noindex or exclude before launch ([SEO.md](./SEO.md)).

**Phase 3 rough completion**

| By tracker remaining lists | ~75% |
| By repo file coverage vs 325 manifest | ~85–92% (after spreadsheet reconciliation) |
| By content parity / QA | ~70% |

---

### Phase 4 — QA & visual polish — **Early / not complete**

This phase is the main gap between “files exist” and “ready for DNS.”

**Not done / in progress:**

- [ ] Sitewide visual consistency across templates (hero images, spacing, typography).
- [ ] Homepage Backstop pass or manual parity vs production ([VISUAL-QA.md](./VISUAL-QA.md)).
- [ ] Incomplete pages (empty podcast embeds, placeholder sections, broken images).
- [ ] `data-section` attributes for Backstop on homepage template (called out in migration tooling docs).
- [ ] Cross-browser and mobile pass on **book-call**, **support-center**, top service pages.
- [ ] Internal link crawl (broken paths, WordPress URLs in body copy).
- [ ] Core Web Vitals spot-check (heavy global CSS — see open technical items).

**Phase 4 rough completion:** **~25–35%**

---

### Phase 5 — DNS cutover — **Not started**

Production traffic is still on **WordPress**. Astro deploys to **staging** on push to `main`.

**Not started:**

- [ ] Production `robots.txt` (staging currently `Disallow: /`).
- [ ] Astro `sitemap.xml` + footer sitemap link (still points at WordPress).
- [ ] GTM / analytics in `BaseLayout` (production uses `GTM-PDGND8M` on WP today).
- [ ] DNS / Cloudflare routing to Astro origin.
- [ ] Full run-through of [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md).

**Phase 5 rough completion:** **0%**

---

## Open technical items (tracker + repo verification)

| Item | Tracker / team note | Repo reality (2026-05-18) | Status |
|------|---------------------|---------------------------|--------|
| **Book-a-call page** | Placeholder; needs content | **`src/pages/book-call.astro` is a full custom page** — copy, testimonials, trust blocks, **LeadConnector iframe** + `msgsndr` script. `BookACallTemplate` example route is still a simpler fallback. | **Substantially done** — verify on staging + GHL domain allowlist; update tracker wording. |
| **Auto-deploy on push** | Cloudflare build hook not in VPS `.env` | **GitHub Actions** deploys on every push to `main` (`.github/workflows/deploy.yml` → `wrangler pages deploy`). No `wrangler.toml`; uses `CLOUDFLARE_API_TOKEN`. | **Done for this repo** — clarify if “VPS `.env`” refers to a **separate WordPress/hosting** pipeline; that is orthogonal to `con-staging` CI. |
| **PurgeCSS before cutover** | Templates bundle ~450KB inline CSS/page | **`public/css/production-extracted.css` alone is ~488KB** on every page via `BaseLayout`; plus template `<style>` blocks. **No PurgeCSS** script or config in repo. | **Not started** — plan CSS diet or scoped extracts per template ([STYLING.md](./STYLING.md)). |
| **GTM / analytics** | (Often grouped with cutover) | Not in `BaseLayout.astro`. | **Not started** |
| **Staging basic auth** | — | `functions/_middleware.ts` on Cloudflare Pages. | **Implemented** (env vars in Cloudflare dashboard) |
| **CoreTemplate `<title>` bug** | — | Affects about, FAQ, privacy, blog index, support-center. | **Open** |
| **Migration scripts path** | — | Some scripts still reference hardcoded paths from another machine. | **Open** — fix before bulk regen |

---

## Overall progress dashboard

```
Phase 1  Inventory          ████████████████████  100%
Phase 2  Templates           ████████████████████  100%  (+ extra templates in repo)
Phase 3  Content in repo     █████████████████░░░   85%  (tracker says ~75%; repo ahead on blog/city/case/podcast)
Phase 3  Content parity/QA   ██████████████░░░░░░   70%  (services + spreadsheet sync + images)
Phase 4  Visual / QA         ███████░░░░░░░░░░░░░   30%
Phase 5  DNS cutover         ░░░░░░░░░░░░░░░░░░░░    0%
────────────────────────────────────────────────────────────
Launch-ready (holistic)      ████████████░░░░░░░░   55–65%
```

---

## Recommended next actions (priority order)

See **[TODO.md](./TODO.md)** for the full checklist (specific pages, bulk categories, and launch phases).

1. **Reconcile the Google Sheet** with [PAGE-INVENTORY.md](./PAGE-INVENTORY.md) (especially blog, city, service tabs).
2. **Close the service-page gap** — export Service Pages tab; build or redirect missing URLs.
3. **Phase 4 pass** — homepage + book-call + flagship services; fix empty podcast embeds and `wp-content` hotlinks.
4. **Open technical items** — GTM in `BaseLayout`, `CoreTemplate` title fix, PurgeCSS / CSS strategy, `/examples` noindex.
5. **Phase 5 prep** — [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) owners for DNS, GHL, ClickUp, Search Console.

---

## How to refresh this doc

Re-run a page count from repo root:

```bash
find src/pages -name '*.astro' | wc -l
find src/pages/blog -maxdepth 1 -name '*.astro' ! -name 'index.astro' | wc -l
find src/pages -maxdepth 1 -name 'attorney-marketing-*.astro' | wc -l
find src/pages -maxdepth 1 -name 'law-firm-marketing-*.astro' | wc -l
find src/pages/case-study -name '*.astro' | wc -l
find src/pages/podcasts -maxdepth 1 -name '*.astro' ! -name 'index.astro' | wc -l
```

Compare totals to the migration tracker and update the tables above.

---

## Related documentation

| Doc | Use for |
|-----|---------|
| [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) | Cutover runbook |
| [COMPREHENSIVE-OVERVIEW.md](./COMPREHENSIVE-OVERVIEW.md) | New developer context |
| [MIGRATION-TOOLING.md](./MIGRATION-TOOLING.md) | `build_blog_posts.py`, `build_astro_post.py`, Backstop |
| [VISUAL-QA.md](./VISUAL-QA.md) | Phase 4 homepage screenshots |
| [INTEGRATIONS.md](./INTEGRATIONS.md) | GHL, ClickUp, GTM gap |
| [SEO.md](./SEO.md) | Canonicals, robots, sitemap |
