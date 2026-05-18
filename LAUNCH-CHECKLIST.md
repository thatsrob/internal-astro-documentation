# Launch Checklist

Master checklist for moving **goconstellation.com** from WordPress/Divi to the Astro static site. Use this document during final QA and on cutover day.

**Detailed guides:** [DEPLOYMENT.md](./DEPLOYMENT.md) · [SEO.md](./SEO.md) · [INTEGRATIONS.md](./INTEGRATIONS.md) · [VISUAL-QA.md](./VISUAL-QA.md)

---

## At a glance

| Item | Current state |
|------|----------------|
| **Live site today** | WordPress on production domain |
| **Astro staging** | Cloudflare Pages project `con-staging` (deploy on push to `main`) |
| **~400 pages** | Static HTML in `dist/` |
| **Primary conversion** | `/book-call/` (LeadConnector / GoHighLevel) |
| **Analytics** | GTM on WordPress; **not yet in Astro** |
| **Staging `robots.txt`** | `Disallow: /` (blocks all crawlers) |
| **Staging access** | HTTP Basic Auth via `functions/_middleware.ts` |

Cutover is a **DNS/hosting change**, not only a git merge. Plan who owns DNS, Cloudflare, Search Console, and GHL/ClickUp settings.

---

## Phase 1: Build & deploy readiness

Complete on `main` before scheduling cutover.

### Build and CI

- [ ] `npm run build` passes locally (Node ≥ 22.12)
- [ ] Latest `main` deployed successfully to Cloudflare Pages (`con-staging`)
- [ ] Staging URL reviewed on **real Cloudflare host** (not only `localhost`)
- [ ] No secrets in git (`.env`, API tokens)
- [ ] No accidental large binaries added under `public/`

### Known code fixes (verify or fix before launch)

- [ ] **`CoreTemplate` title bug**: passes `seoTitle` to `BaseLayout` but layout expects `title` (`about-us`, `faq`, `privacy-policy`, `blog/index`, `support-center` may have wrong `<title>`)
- [ ] **`book-call.astro`**: confirm full SEO props in view-source
- [ ] **`/examples/*` routes**: noindex, remove from build, or block in `robots.txt` before production

### Visual QA (optional but recommended)

- [ ] Homepage Backstop pass or manual side-by-side vs production ([VISUAL-QA.md](./VISUAL-QA.md))
- [ ] Spot-check city page, service page, blog post, podcast page on staging

---

## Phase 2: Content & URLs

### Redirects

- [ ] All WordPress URL changes captured in `astro.config.mjs` `redirects` (today: 3 long-slug → short-slug rules)
- [ ] Cloudflare **bulk redirects** prepared for any URLs not in Astro config
- [ ] Trailing-slash behavior consistent (most production URLs use trailing `/`)
- [ ] Test redirects on staging: old URL → new URL (301/302 as intended)

### Canonicals & meta

- [ ] **Primary host decided:** `www.goconstellation.com` vs `goconstellation.com` (align DNS, redirects, and canonicals)
- [ ] Top 20 URLs have correct `canonicalUrl`, `seoTitle`, `seoDescription` in view-source
- [ ] High-intent pages verified: `/`, `/book-call/`, `/law-firm-seo-services/`, `/case-studies/`, major practice areas
- [ ] One H1 per page; images have `alt` text on key templates

### Internal links & assets

- [ ] Header/footer nav works on staging (all dropdown links)
- [ ] Footer **sitemap link** updated (today points at WordPress `sitemap_index.xml`)
- [ ] Critical images **not** dependent on live WordPress (`wp-content/uploads` URLs migrated to `public/images/` where needed: especially `book-call` testimonials, homepage badges)
- [ ] No indexable pages still relying on **Ninja Forms** / `wp-admin/admin-ajax.php`

---

## Phase 3: Integrations

Test on the **staging hostname** before DNS switch.

### Must work

- [ ] **`/book-call/`**: LeadConnector iframe loads; test booking (widget `7WWVTTJsRrDeHG7AJqpY`)
- [ ] **`/support-center/`**: ClickUp form iframe loads and submits
- [ ] **Sample podcast**: YouTube embed plays
- [ ] **`/thank-you/`**: loads after booking flow (confirm GHL redirect URL)
- [ ] Social links in footer open correct profiles

### Must configure before WordPress shutdown

- [ ] **GTM `GTM-PDGND8M`** (or replacement) added to `BaseLayout` / shared analytics component ([INTEGRATIONS.md](./INTEGRATIONS.md))
- [ ] GTM preview mode tested on Astro host; key tags (GA4, ads, etc.) fire
- [ ] **GHL**: allowed domains include production hostname
- [ ] **ClickUp**: form still accepts submissions from production domain
- [ ] Cookie/consent approach decided if required (not implemented in Astro today)

---

## Phase 4: SEO & discoverability

### Production-only file changes

- [ ] Replace `public/robots.txt`: remove `Disallow: /`; add `Allow: /` and sitemap URL
- [ ] Generate **`sitemap.xml`** (add `@astrojs/sitemap` or build script) and host at `/sitemap.xml`
- [ ] Update footer sitemap href to Astro sitemap
- [ ] Optional: `noindex` for `/examples/`, thank-you, or utility pages per strategy

### Search Console & monitoring

- [ ] Google Search Console property confirmed for chosen primary host
- [ ] Sitemap submitted after launch
- [ ] URL inspection on homepage and 2–3 money pages post-launch
- [ ] Plan to monitor Coverage / Crawl errors for 2–4 weeks

### Social / structured data (nice-to-have)

- [ ] OG images use **absolute** URLs if share previews matter
- [ ] Optional: sitewide `Organization` JSON-LD in `BaseLayout`
- [ ] Optional: `Article` schema on key blog posts

---

## Phase 5: Hosting & DNS (cutover day)

### Pre-switch (same day, before DNS)

- [ ] Final `npm run build` on release commit
- [ ] Deploy to target production Pages project (same `con-staging` or new `con-production`)
- [ ] **Remove or relax staging Basic Auth** on production project (keep auth only on a separate staging domain if needed)
- [ ] Production `robots.txt` and sitemap deployed in `public/`
- [ ] WordPress kept available **read-only** for quick rollback (24–72 hours)
- [ ] Team aligned on rollback owner and DNS revert steps

### DNS / Cloudflare

Pick one approach ([DEPLOYMENT.md](./DEPLOYMENT.md)):

| Approach | Action |
|----------|--------|
| **Same Pages project** | Point `www` / apex custom domain at `con-staging` (or renamed project) |
| **New Pages project** | Create `con-production`; update GitHub workflow `project-name` |
| **Phased** | Cloudflare redirect rules: % traffic or path-based to Astro |

Checklist:

- [ ] DNS records updated (A/CNAME to Cloudflare Pages)
- [ ] SSL certificate active on production domain
- [ ] Apex → `www` redirect (or reverse), per chosen canonical host
- [ ] Low TTL set **before** switch; restored after stable

### Post-switch smoke test (within 1 hour)

Open each on **production domain** (hard refresh / incognito):

| URL | Check |
|-----|--------|
| `/` | Homepage renders; nav works |
| `/book-call/` | Calendar embed loads |
| `/support-center/` | ClickUp form loads |
| `/law-firm-seo-services/` | Flagship service page |
| `/case-studies/` | Results hub |
| `/blog/` | Blog index |
| One blog post | Content + canonical in source |
| One city page | e.g. `/attorney-marketing-austin/` |
| One podcast | Video embed |
| One legacy URL | Redirect from old WordPress slug |

- [ ] GTM / analytics receiving traffic (real-time debug)
- [ ] No spike in 404s in Cloudflare analytics
- [ ] Forms: test booking + support ticket on production

---

## Phase 6: First week after launch

- [ ] Search Console: sitemap processed, no critical indexing errors
- [ ] Crawl top 50 landing pages (Screaming Frog or GSC export) for broken links/images
- [ ] Monitor form submissions and call volume (GHL dashboard)
- [ ] Check Core Web Vitals / PageSpeed on homepage and book-call
- [ ] Confirm WordPress can be decommissioned (or parked) after stable period
- [ ] Document any post-launch redirect additions in `astro.config.mjs`

---

## Rollback plan

If critical issues appear after DNS switch:

| Step | Action |
|------|--------|
| **Fast** | Cloudflare Pages → promote **previous deployment** |
| **DNS** | Point domain back to WordPress origin |
| **Git** | Revert merge on `main` and redeploy if needed |

- [ ] Rollback owner named before cutover
- [ ] WordPress origin still reachable until sign-off

---

## Owners (fill in)

| Area | Owner | Notes |
|------|--------|-------|
| DNS / Cloudflare | | |
| GitHub / deploy | | |
| SEO / Search Console | | |
| GTM / analytics | | |
| GHL / book-call | | |
| ClickUp / support | | |
| Content sign-off | | |

---

## Reference: critical files

| File | Why it matters at launch |
|------|---------------------------|
| `astro.config.mjs` | `site`, `redirects` |
| `public/robots.txt` | Indexing on/off |
| `src/layouts/BaseLayout.astro` | Meta, future GTM |
| `src/pages/book-call.astro` | Primary conversion |
| `src/pages/support-center.astro` | Support form |
| `functions/_middleware.ts` | Staging basic auth |
| `.github/workflows/deploy.yml` | Deploy pipeline |

---

## Related documentation

| Document | Use for |
|----------|---------|
| [DEPLOYMENT.md](./DEPLOYMENT.md) | CI/CD, Wrangler, hosting options, rollback |
| [SEO.md](./SEO.md) | Canonicals, sitemaps, robots, meta |
| [INTEGRATIONS.md](./INTEGRATIONS.md) | GHL, ClickUp, YouTube, GTM gap |
| [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) | Page authoring, URL patterns |
| [VISUAL-QA.md](./VISUAL-QA.md) | Backstop homepage QA |
| [SITE-OVERVIEW.md](./SITE-OVERVIEW.md) | Architecture and scale |
