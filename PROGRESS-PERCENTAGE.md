# Progress by Percentage

**Last updated:** 2026-05-18  
**Purpose:** Percent-complete view by category, what remains for **100% deployment readiness**, and estimated timeline.

**Related:** [PROGRESS.md](./PROGRESS.md) · [QA-BREAKDOWN.md](./QA-BREAKDOWN.md) · [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) · [Migration tracker (Google Sheet)](https://docs.google.com/spreadsheets/d/13jFsEzuXJp5MbTvwvQ1JmuTFIeLlGJ3zLXCSAUlMf4A/edit?usp=sharing)

---

## Executive summary

| Metric | Estimate |
|--------|----------|
| **Migration tracker (325 URLs)** | Team sheet ~**75%** (often stale vs repo) |
| **Files in repo** | **396** routes (**386** excl. `/examples/*`) |
| **Overall deployment readiness** | **~55–60%** |
| **Production today** | WordPress at goconstellation.com |
| **Astro staging** | Cloudflare Pages `con-staging` → staging.goconstellation.com |

**Stack:** Astro 6 (static) · Tailwind 4 · Divi legacy CSS · Cloudflare Pages · GitHub Actions on `main`  
**Deploy:** Push to `main` → `npm run build` → `wrangler pages deploy dist --project-name con-staging`

---

## How to read the percentages

Each category uses three lenses:

| Lens | Meaning |
|------|--------|
| **Built** | `.astro` exists, template applied, `npm run build` succeeds |
| **QA / polish** | Matches production intent—visual, links, embeds, meta checked on staging |
| **Deploy-ready** | Built + QA + category-specific requirements for production cutover |

**Deploy-ready** is what matters for launch. **Built** alone overstates readiness when QA and launch config are incomplete.

---

## Overall deployment readiness

Weighted by business impact (~55–60% today):

| Bucket | Weight | Deploy-ready | Contribution |
|--------|--------|--------------|--------------|
| Homepage + book-call + top 3 services | 25% | ~50% | ~12.5% |
| Remaining services + practice areas | 15% | ~45% | ~6.8% |
| Blog + city (volume SEO) | 25% | ~49% | ~12.3% |
| Case study + podcast + generic | 15% | ~54% | ~8.1% |
| Cross-cutting (GTM, DNS, robots, redirects, assets) | 20% | ~25% | ~5.0% |
| Phase 4 sign-off / crawl / mobile | 10% | ~30% | ~3.0% |
| **Total** | 100% | — | **~48–55%** |

```
Phase 1  Inventory           ████████████████████  100%
Phase 2  Templates          ████████████████████  100%
Phase 3  Content (built)    █████████████████░░░   88%
Phase 3  Content (parity)   ██████████████░░░░░░   70%
Phase 4  QA & polish         ███████░░░░░░░░░░░░░   30%
Phase 5  DNS / launch        ░░░░░░░░░░░░░░░░░░░░    0%
────────────────────────────────────────────────────────
Deployment readiness        ████████████░░░░░░░░   55%
```

---

## By category

### Core marketing & conversion

| Category | Target | In repo | Built | QA | Deploy-ready | To reach 100% |
|----------|--------|---------|------:|---:|-------------:|-------------|
| **Homepage** | 1 | 1 | 95% | 25% | **45%** | `data-section` on `HomepageTemplate`; Backstop/manual parity vs WP; confirm v1 not v2/v3; mobile |
| **Book-a-call** | 1 | 1 | 100% | 50% | **65%** | GHL on staging + prod domain; mobile layout; migrate WP testimonial images |
| **Thank-you** | 1 | 1 | 100% | 40% | **60%** | GHL redirect URL; meta tags; post-booking flow |
| **Services** | ~18 | 9 | 50% | 30% | **40%** | Build missing URLs (~9); QA 3 flagship + 6 `ServiceTemplate` pages |
| **Practice areas** | 12 | 12 | 100% | 40% | **55%** | Visual/content sample per area; canonicals; images |

**Services in repo today (9):**

| Page | Type |
|------|------|
| `law-firm-seo-services`, `law-firm-ppc`, `law-firm-web-design` | Custom flagship |
| `meta-ads`, `law-firm-branding`, `email-marketing-for-law-firms`, `social-media-for-law-firms`, `ai-chatbot-for-law-firms`, `bilingual-seo-law-firms` | `ServiceTemplate` |

---

### Content at scale

| Category | Target (tracker) | In repo | Built | QA | Deploy-ready | To reach 100% |
|----------|------------------|---------|------:|---:|-------------:|-------------|
| **Blog posts** | ~160–275* | 160 | 95% | 35% | **50%** | Reconcile sheet; sample 25 posts; WP hotlinks; `blog/index` list |
| **City pages** | ~117–121 | 121 | 98% | 30% | **48%** | Sample 8–10 vs production; `CityTemplate` WP image URLs; canonicals |
| **Case studies** | ~21–27 | 27 | 100% | 40% | **55%** | Narrative QA; `case-studies` hub links |
| **Podcasts** | ~21–23 | 23 | 100% | 45% | **58%** | Fix empty `embedUrl` on `how-to-close-more-cases`; test embeds on staging |

\*Tracker still lists **115 blogs remaining** while repo has **160** files—update the sheet before trusting build %.

**Blog template mix (repo):**

| Template | Count |
|----------|------:|
| `BlogPostTemplate` | 129 |
| `AgencyReviewTemplate` | 11 |
| `GuidesTemplate` | 3 |
| `PodcastTemplate` (under `/blog/`) | ~10 |
| Other / mixed | ~17 |

---

### Auxiliary / generic pages (tracker “12 generic”)

| Group | Count | Built | QA | Deploy-ready | To reach 100% |
|-------|------:|------:|---:|-------------:|-------------|
| **Core** (about, FAQ, privacy, terms, support) | 5 | 100% | 45% | **55%** | Fix `CoreTemplate` `seoTitle` → `title` bug; ClickUp on support |
| **Hubs** (services, case-studies) | 2 | 100% | 40% | **55%** | IA and links match production |
| **Partner / referral** (become-a-partner, partner-program, referral-program) | 3 | 100% | 35% | **50%** | Content and CTA QA |
| **Custom** (patrick-carver, lead-flow-mastery-guide) | 2 | 100% | 30% | **45%** | Confirm in manifest; content QA |
| **All generic** | **13** | **100%** | **40%** | **52%** | — |

**Generic pages (all files exist):**  
`about-us`, `faq`, `privacy-policy`, `terms-of-service`, `support-center`, `services`, `case-studies`, `book-call`, `thank-you`, `become-a-partner`, `partner-program`, `referral-program`, `patrick-carver`, `lead-flow-mastery-guide`

---

### Nested & dev-only routes

| Category | In repo | Built | QA | Deploy-ready | To reach 100% |
|----------|--------|------:|---:|-------------:|-------------|
| **`law-firm-seo/*` guides** | 13 | 100% | 25% | **40%** | Include in blog/service QA sample |
| **`lawyer-advertising/*`** | 3 | 100% | 25% | **40%** | Quick parity check |
| **`law-firm-web-design/*` sub** | 1 | 100% | 25% | **40%** | Tie to web-design service QA |
| **`/examples/*`** | 10 | 100% | — | **0%** (prod) | **noindex, remove, or exclude from build** |

---

### Cross-cutting (all categories)

| Area | Deploy-ready | To reach 100% |
|------|-------------:|-------------|
| **Templates (13 in repo)** | 70% | `CoreTemplate` title fix; homepage `data-section`; v2/v3 cleanup |
| **Redirects** | 25% | Full manifest map in `astro.config.mjs` + Cloudflare bulk rules (only 3 today) |
| **SEO / discoverability** | 20% | Production `robots.txt`; `sitemap.xml`; footer sitemap link (WP today) |
| **Analytics (GTM)** | 0% | Add `GTM-PDGND8M` to `BaseLayout`; preview on staging |
| **Assets (WP CDN)** | 60% | Migrate `wp-content/uploads` hotlinks on P0 pages |
| **Performance / CSS** | 40% | PurgeCSS or CSS diet (~488KB `production-extracted.css`) |
| **Hosting / DNS** | 50% | Production domain → Astro; staging auth off prod; smoke tests |

---

## One-line category summary

| Category | Deploy-ready now | Main gap to 100% |
|----------|------------------|------------------|
| Homepage | 45% | Visual parity, `data-section`, mobile |
| Book-call / thank-you | 60–65% | Widget + domain testing, images |
| Services | 40% | ~50% of manifest URLs still missing in repo |
| Practice areas | 55% | QA (files done) |
| Blog | 50% | Sheet reconcile + sampled QA |
| Cities | 48% | Sample QA + WP images in template |
| Case studies | 55% | Content QA |
| Podcasts | 58% | One empty embed + staging tests |
| Generic (13) | 52% | CoreTemplate titles, support form |
| Launch infra | 25% | GTM, DNS, robots, sitemap, redirects |

---

## 100% deployment readiness — global checklist

**Definition:** Safe to switch DNS—not necessarily every page pixel-perfect.

### Blockers (must be 100%)

- [ ] Production DNS → Astro on Cloudflare
- [ ] P0 functional: `/`, `/book-call/`, `/support-center/`, `/thank-you/` on production host
- [ ] GTM / analytics on Astro (`GTM-PDGND8M`)
- [ ] Production `robots.txt` + `sitemap.xml` + Search Console submit
- [ ] Redirect map — no major 404s on top 50 URLs
- [ ] GHL + ClickUp production domain allowlists
- [ ] `/examples/*` not indexable
- [ ] `npm run build` + CI green on `main`
- [ ] `CoreTemplate` → `BaseLayout` `title` fix
- [ ] P0 images not dependent on live WordPress `wp-content`
- [ ] Every manifest service URL built or redirected
- [ ] Phase 4 sign-off on staging ([QA-BREAKDOWN.md](./QA-BREAKDOWN.md))

### Strongly recommended

- [ ] Top 20 URLs meta/canonical audit
- [ ] Homepage within agreed visual threshold
- [ ] Sample QA: 10 cities, 15 blogs, podcasts, top case studies
- [ ] Link crawl with documented exceptions
- [ ] Mobile pass on P0 pages

### Acceptable fast-follow

- [ ] PurgeCSS / full CSS optimization
- [ ] Full-site Backstop
- [ ] WCAG audit
- [ ] Manual review of all 160 blog posts
- [ ] Article schema on all posts

---

## Estimated timeline to 100% deployment readiness

**Assumption:** 1 FTE dev + part-time QA (~20 hrs/week). Scale down duration with more people.

| Phase | Work | Duration |
|-------|------|----------|
| **A. Scope lock** | Reconcile Google Sheet vs repo; list missing service URLs | 2–4 days |
| **B. Service gap** | Build ~9 missing service pages or redirects | 1–2 weeks |
| **C. Launch blockers** | GTM, robots, sitemap, CoreTemplate, examples noindex, P0 images, redirects | 1 week |
| **D. Phase 4 QA** | P0 visual + functional; homepage; sample blog/city/podcast | 2–3 weeks |
| **E. Sign-off** | Link crawl, top-20 meta, GHL/ClickUp prod, runbook | 3–5 days |
| **F. Cutover** | DNS, smoke tests, GSC, 48h monitoring | 2–3 days |

| Scenario | Calendar | Team |
|----------|----------|------|
| **Aggressive** | 3–4 weeks | 2 devs + dedicated QA week 3; small service gap after reconcile |
| **Realistic** | 5–7 weeks | 1 dev + part-time QA |
| **Conservative** | 8–10 weeks | Part-time dev; full homepage + all-service visual parity; PurgeCSS pre-launch |

**Critical path:** Service manifest closure **(B)** + P0 QA on staging **(D)** in parallel → launch config **(C)** → cutover **(F)**.  
Blog/city volume is **not** on the critical path if sampled QA passes and redirects cover URL mismatches.

---

## Refreshing these numbers

```bash
# Page counts
find src/pages -name '*.astro' | wc -l
find src/pages/examples -name '*.astro' | wc -l
find src/pages/blog -maxdepth 1 -name '*.astro' ! -name 'index.astro' | wc -l
find src/pages -maxdepth 1 -name 'attorney-marketing-*.astro' | wc -l
find src/pages -maxdepth 1 -name 'law-firm-marketing-*.astro' | wc -l
find src/pages/case-study -name '*.astro' | wc -l
find src/pages/podcasts -maxdepth 1 -name '*.astro' ! -name 'index.astro' | wc -l

# WP image hotlinks
grep -r "wp-content/uploads" src/ --include="*.astro" -l | wc -l
```

Update this doc after spreadsheet reconcile or major migration merges.

---

## Related documentation

| Document | Use for |
|----------|---------|
| [PROGRESS.md](./PROGRESS.md) | Phase status and repo audit tables |
| [QA-BREAKDOWN.md](./QA-BREAKDOWN.md) | Phase 4 workstreams and sign-off |
| [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) | Cutover runbook |
| [INTEGRATIONS.md](./INTEGRATIONS.md) | GHL, ClickUp, GTM |
| [SEO.md](./SEO.md) | Canonicals, robots, sitemap |
| [COMPREHENSIVE-OVERVIEW.md](./COMPREHENSIVE-OVERVIEW.md) | Stack and architecture |
