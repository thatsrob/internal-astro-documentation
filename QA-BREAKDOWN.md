# Phase 4 — QA & Polish Breakdown

**Purpose:** Define everything needed to move from “pages exist in the repo” (Phase 3) to “safe to cut over” (Phase 5). Phase 4 is **quality assurance and polish**—not net-new feature work.

**Related docs:** [PROGRESS.md](./PROGRESS.md) · [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) · [VISUAL-QA.md](./VISUAL-QA.md) · [INTEGRATIONS.md](./INTEGRATIONS.md) · [SEO.md](./SEO.md)

**Estimated phase completion today:** ~25–35% ([PROGRESS.md](./PROGRESS.md))

---

## What Phase 4 is (and is not)

| Phase 4 **is** | Phase 4 **is not** |
|----------------|-------------------|
| Verifying pages match production intent | Writing ~300 pages from scratch (most files already exist) |
| Fixing broken images, links, embeds, titles | DNS cutover or production deploy |
| Visual parity on high-traffic templates | Adding GTM/sitemap (launch config—often done in parallel with Phase 5 prep) |
| Mobile, cross-browser, and performance spot-checks | Automated test suite for every URL (no full E2E suite in repo today) |
| Sign-off on money pages before launch | Reconciling the migration tracker spreadsheet (Phase 3 admin, but blocks QA scope) |

**Exit criterion:** A named owner can run the [Phase 4 sign-off checklist](#phase-4-sign-off-checklist) on **staging.goconstellation.com** (production build on Cloudflare, not only `localhost`) and record pass/fail.

---

## How Phase 4 fits in the migration

```
Phase 3  Content exists in Git        →  mostly done (~85% files)
Phase 4  QA & polish (this doc)       →  in progress (~25–35%)
Phase 5  DNS cutover + launch config  →  not started
```

Phase 4 assumes:

1. `npm run build` passes on `main`.
2. Staging deploy is current (`con-staging` on Cloudflare).
3. Migration tracker is reconciled with repo so QA knows the **official URL list** ([Google Sheet](https://docs.google.com/spreadsheets/d/13jFsEzuXJp5MbTvwvQ1JmuTFIeLlGJ3zLXCSAUlMf4A/edit?usp=sharing)).

---

## QA workstreams

Phase 4 splits into seven workstreams. Use **P0 / P1 / P2** priorities below.

### 1. Visual & layout parity

**Goal:** Staging looks like production on templates that drive brand trust and conversion.

| Task | Priority | Notes |
|------|----------|--------|
| Homepage vs production (section by section) | **P0** | Backstop or manual side-by-side |
| Add `data-section` attributes to `HomepageTemplate.astro` | **P0** | Required for Backstop scroll targets; **not present today** |
| Book-call, law-firm-seo-services, law-firm-ppc, law-firm-web-design | **P0** | Custom flagship layouts |
| One city page + one practice area + one service (`ServiceTemplate`) | **P1** | Representative template QA |
| Blog post + agency review + guide | **P1** | `BlogPostTemplate`, `AgencyReviewTemplate`, `GuidesTemplate` |
| Podcast episode page | **P1** | Video embed + platform links |
| Case study narrative | **P1** | `CaseStudyNarrativeTemplate` |
| Core pages (about, FAQ, privacy, blog index) | **P1** | `CoreTemplate` |
| Sitewide consistency (spacing, type, green/navy tokens) | **P2** | Cross-template drift |

**Automated homepage tool:** [VISUAL-QA.md](./VISUAL-QA.md) — Backstop compares production vs `/examples/homepage-example/` at **1440×900 desktop only**.

**Manual checks per template:**

- [ ] Hero: image loads, headline readable, CTA visible
- [ ] Nav dropdowns not clipped; mobile menu works
- [ ] Footer alignment and social icons
- [ ] No obvious overflow / horizontal scroll at 375px, 768px, 1440px
- [ ] Fonts: Poppins headings load (not fallback serif everywhere)

**Known visual debt:**

- ~488KB `production-extracted.css` on every page ([STYLING.md](./STYLING.md))
- Mix of Divi classes, Tailwind, and inline styles
- PurgeCSS / CSS diet planned pre-cutover—not Phase 4 blocker unless performance fails targets

---

### 2. Functional & integration QA

**Goal:** Conversion paths and third-party embeds work on the **staging hostname** (widgets may block `localhost`).

| Page / flow | Priority | What to verify |
|-------------|----------|----------------|
| `/book-call/` | **P0** | LeadConnector iframe loads; calendar books; mobile scroll/height OK |
| `/support-center/` | **P0** | ClickUp form loads and submits |
| `/thank-you/` | **P0** | Loads; GHL redirect URL correct in GoHighLevel settings |
| Header “Book a Call” CTA | **P0** | Links to `/book-call/` |
| Footer social links | **P1** | Facebook, Instagram, LinkedIn, X, YouTube |
| Podcast YouTube embed | **P1** | Sample 3 episodes; fix empty `embedUrl` where needed |
| Blog posts with video `embedUrl` | **P2** | Spot-check posts using `PodcastTemplate` on blog paths |

**Known issue:** `src/pages/podcasts/how-to-close-more-cases.astro` has `embedUrl=""` (placeholder player).

**Not functional on Astro (verify absent or removed from indexable pages):**

- Ninja Forms / `wp-admin/admin-ajax.php` from migrated WordPress HTML
- WordPress-only shortcodes in body copy

**Test environment:**

- [ ] Run tests on **staging.goconstellation.com** with basic auth credentials (if enabled)
- [ ] Re-test top flows after any Cloudflare or GHL domain allowlist change

Details: [INTEGRATIONS.md](./INTEGRATIONS.md)

---

### 3. Content & asset parity

**Goal:** Copy, images, and media match production (or approved updates)—no silent regressions.

| Task | Priority | Notes |
|------|----------|--------|
| Reconcile tracker URL list vs repo | **P0** | Scope for “done” |
| Service pages: manifest vs built pages | **P0** | Main content gap vs tracker |
| Replace `wp-content/uploads` hotlinks | **P0** | ~15+ files in `src/` still reference live WP CDN; breaks when WP shuts down |
| Migrate book-call testimonial photos to `public/images/` | **P0** | 4 URLs on `book-call.astro` still on WP |
| CityTemplate case-study card images (6 URLs in template) | **P1** | Hardcoded `wp-content` in `CityTemplate.astro` |
| Blog body: broken images, lazy-load placeholders | **P1** | Some posts retain WP lazy-load SVG placeholders |
| Blog index manual list vs all posts | **P1** | New posts may be missing from `blog/index.astro` |
| Podcast show notes completeness | **P2** | Typos, broken internal links |
| `blog/index` lists only featured posts | **P2** | Intentional? confirm with stakeholders |

**Asset audit command (find WP hotlinks in source):**

```bash
grep -r "wp-content/uploads" src/ --include="*.astro" -l
```

**Content sampling strategy (don’t open 400 pages):**

| Bucket | Sample size | Method |
|--------|-------------|--------|
| City pages | 5–10 | Top traffic cities + one random |
| Blog posts | 10–15 | Top organic URLs from GSC + random |
| Case studies | All 27 or top 10 | Smaller set |
| Podcasts | All 23 or episodes missing embed | Check `embedUrl` in frontmatter |
| Service / practice | All manifest URLs | Checklist against sheet |

---

### 4. SEO & metadata QA

**Goal:** Every indexable page has correct head tags; no staging-only mistakes ship to production.

| Task | Priority | Notes |
|------|----------|--------|
| Fix `CoreTemplate` → `BaseLayout` `title` prop bug | **P0** | Passes `seoTitle` but layout expects `title` (about, FAQ, privacy, blog index, support-center) |
| Top 20 URLs: title, description, canonical | **P0** | View-source on staging |
| `www` vs non-`www` canonical consistency | **P0** | Align with DNS decision |
| `/examples/*` — noindex or remove from build | **P0** | 10 routes ship today |
| One H1 per page | **P1** | Especially migrated blog HTML |
| Trailing slash on canonicals | **P1** | Match production |
| OG/Twitter images (absolute URLs) | **P2** | Share debugger on homepage + book-call |

**Top 20 URL list (minimum):**

`/`, `/book-call/`, `/law-firm-seo-services/`, `/law-firm-ppc/`, `/law-firm-web-design/`, `/case-studies/`, `/about-us/`, `/blog/`, `/services/`, `/support-center/`, plus top practice areas and 5–8 priority blog/city URLs.

Details: [SEO.md](./SEO.md)

---

### 5. Navigation, links & redirects

**Goal:** Users and bots never hit 404s from internal nav or legacy WordPress paths.

| Task | Priority | Notes |
|------|----------|--------|
| Header nav: all dropdown links | **P0** | Practice areas, services, resources |
| Footer links (including legal) | **P0** | |
| Footer sitemap link | **P1** | Still `/sitemap_index.xml` (WordPress)—404 on Astro until fixed |
| Internal links in blog/city body → correct paths | **P1** | Many absolute `goconstellation.com` links (OK) but verify path changes |
| Redirect map: manifest old URLs → Astro | **P0** | Only 3 pairs in `astro.config.mjs` today; rest may need Cloudflare bulk rules |
| Test redirects on staging host | **P0** | Old slug → new slug |
| External links open with `rel="noopener"` where `target="_blank"` | **P2** | Spot-check |

**Crawl options (pick one):**

- Screaming Frog / Sitebulb against staging (export from build: `npm run build && npm run preview`)
- `npx linkinator` or similar on preview URL
- GSC coverage export crossed with manifest

---

### 6. Performance & technical polish

**Goal:** Money pages are acceptably fast; no launch-day surprise from CSS weight.

| Task | Priority | Target / notes |
|------|----------|----------------|
| Lighthouse on `/`, `/book-call/`, one blog | **P1** | LCP, CLS—not formal CI gate today |
| Confirm `production-extracted.css` necessity | **P2** | PurgeCSS or per-template CSS before/after cutover |
| Image format: WebP where migrated | **P2** | `public/images/` |
| Font preload in `BaseLayout` | **P2** | Poppins woff2 |
| Build time at ~400 pages acceptable in CI | **P1** | `npm run build` in GitHub Actions |

**Not in repo yet:** automated performance budgets or PurgeCSS pipeline.

---

### 7. Accessibility & compliance (spot-check)

**Goal:** No obvious a11y blockers on conversion pages; legal pages present.

| Task | Priority | Notes |
|------|----------|--------|
| Keyboard nav: header menu + book-call | **P1** | Focus states on dropdown triggers |
| Form labels / iframe titles | **P1** | ClickUp iframe has `title`; booking iframe has `title` on template variant |
| Color contrast on green CTAs | **P2** | Brand green on white |
| `privacy-policy`, `terms-of-service` linked in footer | **P0** | Legal |
| Cookie/consent banner | **P2** | Not implemented; decide if required before EU traffic |

Full WCAG audit is **out of scope** unless stakeholders require it pre-launch.

---

## Priority summary

### P0 — Must pass before cutover

- [ ] Homepage acceptable vs production (manual or Backstop after `data-section` fix)
- [ ] Book-call + support-center + thank-you flows on staging domain
- [ ] Top 20 URLs metadata + `CoreTemplate` title fix
- [ ] Critical `wp-content` images migrated or confirmed WP stays up until migrated
- [ ] Redirect map for high-traffic legacy URLs
- [ ] `/examples/*` handled (noindex or removed)
- [ ] Header/footer nav complete
- [ ] `npm run build` clean; staging deploy matches `main`

### P1 — Should pass; document exceptions

- [ ] Template sample QA (city, blog, service, practice area, podcast, case study)
- [ ] Mobile layout on P0 pages
- [ ] Blog index vs post inventory
- [ ] Footer sitemap strategy (Astro sitemap or redirect)
- [ ] Link crawl with acceptable 404 list
- [ ] Lighthouse spot-check on money pages

### P2 — Post-launch or fast-follow OK

- [ ] Sitewide visual micro-polish
- [ ] PurgeCSS / CSS optimization
- [ ] OG absolute URLs
- [ ] Full podcast embed audit
- [ ] Structured data enhancements
- [ ] Full accessibility audit

---

## Template QA cheat sheet

Use `/examples/*` routes to preview template chrome without hunting production URLs.

| Template | Example route | Production sample |
|----------|-----------------|-------------------|
| `HomepageTemplate` | `/examples/homepage-example/` | `/` |
| `BlogPostTemplate` | `/examples/blog-post-example/` | Any `src/pages/blog/*.astro` |
| `CityTemplate` | `/examples/city-example/` | e.g. `/attorney-marketing-austin/` |
| `ServiceTemplate` | `/examples/service-example/` | e.g. `/meta-ads/` |
| `PracticeAreaTemplate` | `/examples/practice-area-example/` | e.g. `/immigration-law-firm-marketing/` |
| `PodcastTemplate` | `/examples/podcast-example/` | `/podcasts/...` |
| `CoreTemplate` | `/examples/core-example/` | `/about-us/`, `/faq/` |
| `BookACallTemplate` | `/examples/book-a-call-example/` | **Not** production book-call (custom page) |

**Per-template checklist (copy for each QA ticket):**

```
[ ] View-source: title, description, canonical
[ ] One clear H1
[ ] Hero image / featured image loads
[ ] CTA links correct
[ ] Mobile 375px: no horizontal scroll
[ ] Desktop 1440px: matches design intent
[ ] No console errors (critical)
[ ] No wp-content broken images
```

---

## Tools & commands

| Tool | Use for |
|------|---------|
| `npm run dev` | Fast iteration |
| `npm run build && npm run preview` | Production-like QA |
| BackstopJS | Homepage section diffs — [VISUAL-QA.md](./VISUAL-QA.md) |
| `scripts/fix-loop.mjs` | Optional AI-assisted homepage CSS (needs `ANTHROPIC_API_KEY`) |
| Browser view-source | Meta tags |
| Chrome DevTools device mode | Mobile |
| Screaming Frog / link checker | Site-wide links |
| Google Rich Results Test | JSON-LD spot-check |
| Facebook Sharing Debugger | OG tags |
| GTM Preview | After GTM added to `BaseLayout` |

**Backstop quick start:**

```bash
# Terminal 1
npm run dev

# Terminal 2 — first time or after production homepage change
npx backstop reference --config=backstop.config.cjs
npx backstop test --config=backstop.config.cjs
open backstop_data/html_report/index.html
```

---

## Roles & workflow (suggested)

| Role | Owns |
|------|------|
| **Design / migration** | Homepage + template visual parity, `data-section`, hero images |
| **Dev** | `CoreTemplate` bug, redirects, wp-content migration, examples noindex |
| **Content** | Blog/city copy, blog index, service page gaps vs tracker |
| **Marketing ops** | GHL, ClickUp, GTM (with dev for `BaseLayout`) |
| **QA lead** | Sign-off checklist, crawl 404 report, staging test pass |

**Suggested flow:**

1. Reconcile migration tracker → official URL list.
2. Close P0 functional tests on staging.
3. Run P0 visual on homepage + money pages.
4. Run sampled template QA (spreadsheet with pass/fail/owner).
5. Link crawl + redirect test.
6. Sign-off meeting → Phase 5 scheduling.

---

## Phase 4 sign-off checklist

Run on **https://staging.goconstellation.com** (or current Cloudflare staging URL) after latest `main` deploy.

### Build & deploy

- [ ] `npm run build` passes locally
- [ ] Latest `main` deployed to `con-staging`
- [ ] Tested with basic auth if enabled

### P0 pages (visual + functional)

- [ ] `/` — homepage
- [ ] `/book-call/` — booking widget
- [ ] `/support-center/` — ClickUp form
- [ ] `/thank-you/`
- [ ] `/law-firm-seo-services/`
- [ ] `/law-firm-ppc/` or `/law-firm-web-design/` (at least one flagship service)

### Metadata & SEO prep

- [ ] Top 20 URLs view-source pass
- [ ] `CoreTemplate` titles fixed
- [ ] `/examples/*` not indexable
- [ ] No critical `wp-content` 404s on P0 pages

### Nav & links

- [ ] Full header/footer click-through
- [ ] Redirect test list (documented URLs) passes
- [ ] 404 report reviewed; blockers assigned

### Sign-off

| Field | Value |
|-------|--------|
| **Date** | |
| **Staging build / commit** | |
| **Signed off by** | |
| **Known exceptions (link to tickets)** | |

---

## Known blockers for Phase 4 (repo state)

| Issue | Impact | Owner |
|-------|--------|--------|
| No `data-section` on `HomepageTemplate` | Backstop staging scroll wrong | Dev |
| `CoreTemplate` `seoTitle` vs `title` | Wrong `<title>` on key pages | Dev |
| `wp-content` hotlinks | Images break when WP off | Dev / content |
| Only 1 podcast with empty `embedUrl` | Broken video on that episode | Content |
| Footer sitemap → WordPress | 404 on Astro | Dev |
| `/examples/*` in production build | Unwanted indexable URLs | Dev |
| Homepage Backstop mismatch ~15–25%+ | Visual parity incomplete | Design / dev |

---

## After Phase 4

Phase 4 complete does **not** mean launch. Phase 5 still requires:

- Production `robots.txt` and `sitemap.xml`
- GTM in `BaseLayout`
- DNS cutover and smoke tests

See [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) and [PROGRESS.md](./PROGRESS.md).

---

## Related documentation

| Document | Topics |
|----------|--------|
| [VISUAL-QA.md](./VISUAL-QA.md) | Backstop commands, thresholds, fix-loop |
| [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) | Full cutover including Phase 4 items |
| [INTEGRATIONS.md](./INTEGRATIONS.md) | GHL, ClickUp, GTM |
| [SEO.md](./SEO.md) | Canonicals, robots, sitemap |
| [STYLING.md](./STYLING.md) | CSS layers, PurgeCSS context |
| [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) | Authoring and URL conventions |
| [TEMPLATES.md](./TEMPLATES.md) | Template props and examples |
