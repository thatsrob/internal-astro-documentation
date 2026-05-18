# Post-Migration To-Do

**Last updated:** 2026-05-18  
**Context:** Page migration (Phase 3) is largely **built in repo**; remaining work is **editing, QA, scope reconciliation, and launch**.  
**Scope authority:** [Migration tracker (Google Sheet)](https://docs.google.com/spreadsheets/d/13jFsEzuXJp5MbTvwvQ1JmuTFIeLlGJ3zLXCSAUlMf4A/edit?usp=sharing)  
**Inventory reference:** [PAGE-INVENTORY.md](./PAGE-INVENTORY.md)

---

## How to use this list

| Symbol | Meaning |
|--------|---------|
| **P0** | Blocker for staging sign-off or DNS cutover |
| **P1** | Should complete pre-launch; document exceptions |
| **P2** | Fast-follow after launch acceptable |

Check boxes as you complete work on **staging.goconstellation.com** (not only `localhost`). Procedures: [QA-PROCEDURES.md](./QA-PROCEDURES.md) · [QA-BREAKDOWN.md](./QA-BREAKDOWN.md).

---

## Phase 0 — Scope lock (do first)

- [ ] **P0** Export remaining Google Sheet tabs (Blog, City, Service, Case Study, Podcast, Practice Area) as CSV and reconcile with repo
- [ ] **P0** Update sheet row status for categories where repo is ahead (blog ~160 files, city ~121, podcasts ~23, case studies ~27)
- [ ] **P0** Export **Service Pages** tab → list every manifest URL; mark built vs missing vs redirect-only
- [ ] **P0** Add missing service URLs to § [Missing service URLs](#missing-service-urls) below
- [ ] **P1** Confirm official **325-URL manifest** matches what QA will test

---

## Phase 1 — Global fixes (all pages)

These fixes affect entire categories—do before wide content editing.

| Task | Priority | Files / area |
|------|----------|----------------|
| Fix `CoreTemplate` → `BaseLayout` `title` prop (`seoTitle` → `title`) | **P0** | `src/layouts/templates/CoreTemplate.astro` |
| Add GTM `GTM-PDGND8M` to `BaseLayout` | **P0** | `src/layouts/BaseLayout.astro` |
| Production `robots.txt` + `sitemap.xml` | **P0** | `public/robots.txt` + sitemap integration |
| `/examples/*` noindex or exclude from production build | **P0** | 10 routes under `src/pages/examples/` |
| Expand redirects beyond 3 pairs in `astro.config.mjs` | **P0** | Top 50 GSC URLs + manifest old slugs |
| Footer sitemap link (still WordPress `sitemap_index.xml`) | **P1** | `SiteFooter.astro` |
| PurgeCSS / CSS diet plan (~488KB Divi extract) | **P2** | `public/css/production-extracted.css` |
| Migrate `CityTemplate` hardcoded `wp-content` card images (6 URLs) | **P1** | `CityTemplate.astro` |

---

## Phase 2 — Pages that need editing (specific)

### P0 — Money & conversion pages

| URL | Page file | Edit / QA focus |
|-----|-----------|-----------------|
| `/` | `src/pages/index.astro` | Confirm `HomepageTemplate` (not v2/v3); add `data-section` for Backstop; visual parity vs WP |
| `/book-call/` | `src/pages/book-call.astro` | Migrate 4 WP testimonial images to `public/images/`; GHL iframe on staging domain; mobile height |
| `/thank-you/` | `src/pages/thank-you.astro` | Meta tags; confirm GHL post-booking redirect URL |
| `/support-center/` | `src/pages/support-center.astro` | ClickUp iframe submit on staging; fix `<title>` via CoreTemplate fix |
| `/law-firm-seo-services/` | `src/pages/law-firm-seo-services.astro` | Full visual QA; canonical; replace WP hotlinks |
| `/law-firm-ppc/` | `src/pages/law-firm-ppc.astro` | Full visual QA; canonical; replace WP hotlinks |
| `/law-firm-web-design/` | `src/pages/law-firm-web-design.astro` | Full visual QA; canonical; replace WP hotlinks |

### P0 — Core / hub pages (`CoreTemplate` + custom hubs)

| URL | Page file | Edit / QA focus |
|-----|-----------|-----------------|
| `/about-us/` | `about-us.astro` | Title/meta after CoreTemplate fix; hero + copy QA |
| `/faq/` | `faq.astro` | Title/meta; accordion sections |
| `/blog/` | `blog/index.astro` | Title/meta; **manual post list** includes new posts |
| `/podcasts/` | `podcasts/index.astro` | Title/meta; episode grid links to all 23 episodes |
| `/privacy-policy/` | `privacy-policy.astro` | Title/meta; legal copy |
| `/terms-of-service/` | `terms-of-service.astro` | Title/meta; legal copy |
| `/case-studies/` | `case-studies.astro` | Custom layout—hub links to all case studies |
| `/services/` | `services.astro` | Custom hub—link every **built** service + plan missing URLs |

### P1 — Service pages (`ServiceTemplate`)

| URL | Page file | Edit / QA focus |
|-----|-----------|-----------------|
| `/meta-ads/` | `meta-ads.astro` | Template sections, CTAs, meta, images |
| `/law-firm-branding/` | `law-firm-branding.astro` | Same |
| `/email-marketing-for-law-firms/` | `email-marketing-for-law-firms.astro` | Same |
| `/social-media-for-law-firms/` | `social-media-for-law-firms.astro` | Same |
| `/ai-chatbot-for-law-firms/` | `ai-chatbot-for-law-firms.astro` | Same |
| `/bilingual-seo-law-firms/` | `bilingual-seo-law-firms.astro` | Same |

### P1 — Partner & utility pages

| URL | Page file | Edit / QA focus |
|-----|-----------|-----------------|
| `/become-a-partner/` | `become-a-partner.astro` | CTA + form/links vs production |
| `/partner-program/` | `partner-program.astro` | Content parity |
| `/referral-program/` | `referral-program.astro` | Content parity |
| `/patrick-carver/` | `patrick-carver.astro` | Bio, images, meta |
| `/lead-flow-mastery-guide/` | `lead-flow-mastery-guide.astro` | Long-form content + internal links |

### P1 — Podcast episode (known defect)

| URL | Page file | Edit / QA focus |
|-----|-----------|-----------------|
| `/podcasts/how-to-close-more-cases/` | `podcasts/how-to-close-more-cases.astro` | **Set `embedUrl`** (currently empty); test YouTube player on staging |

### P2 — Template example routes (dev only)

Do **not** link from production nav. Before launch: noindex or remove from build.

| URL | Purpose |
|-----|---------|
| `/examples/homepage-example/` | Homepage template preview |
| `/examples/blog-post-example/` | Blog template preview |
| `/examples/city-example/` | City template preview |
| `/examples/service-example/` | Service template preview |
| `/examples/practice-area-example/` | Practice area preview |
| `/examples/podcast-example/` | Podcast preview |
| `/examples/core-example/` | Core template preview |
| `/examples/book-a-call-example/` | **Not** production book-call |
| `/examples/homepage-v2-example/` | Migration experiment |
| `/examples/homepage-v3-example/` | Migration experiment |

---

## Phase 3 — Bulk categories (sampled editing)

Do **not** open every file—use sampling + grep-driven fixes.

### Blog posts (~160 files in `src/pages/blog/`)

| Task | Priority | Approach |
|------|----------|----------|
| Reconcile sheet “remaining” vs repo | **P0** | Mark 107→160 complete on sheet or list true gaps |
| Sample QA (10–15 posts) | **P1** | Top GSC URLs + random; check H1, images, internal links |
| Replace `wp-content/uploads` hotlinks | **P0** | `grep -r "wp-content/uploads" src/pages/blog/` |
| Remove dead Ninja Forms / WP shortcodes in body | **P1** | Per sampled posts |
| Fix lazy-load SVG placeholders | **P1** | Migrated Divi artifacts |
| `AgencyReviewTemplate` posts (~11) | **P1** | Review layout + meta |
| `GuidesTemplate` posts (~3) | **P1** | Chapter nav + canonicals |
| Nested guides `law-firm-seo/*` (13) | **P1** | Include in blog/service QA |
| `lawyer-advertising/*` (3) | **P2** | Quick parity |

### City pages (~121 files)

| Task | Priority | Approach |
|------|----------|----------|
| Mark sheet complete (4/118 → ~121) | **P0** | Admin—repo is ahead of sheet |
| Sample QA (8–10 cities) | **P1** | Top traffic + `attorney-marketing-*` + `law-firm-marketing-*` mix |
| Verify `canonicalUrl` vs Astro path | **P1** | Add `astro.config.mjs` redirect if path ≠ canonical |
| Hero images exist under `public/images/` | **P1** | Per sampled cities |
| `CityTemplate` shared WP images | **P1** | Fix once in template (6 URLs) |

### Case studies (~27 in `src/pages/case-study/`)

| Task | Priority | Approach |
|------|----------|----------|
| Reconcile sheet 26/29 vs 27 files | **P1** | Identify 2–3 sheet-only or repo-only URLs |
| Narrative QA | **P1** | Metrics, quotes, images on top 10 |
| Hub links from `/case-studies/` | **P1** | All studies discoverable |

### Podcast episodes (~23 in `src/pages/podcasts/`)

| Task | Priority | Approach |
|------|----------|----------|
| Mark sheet 22/22 complete (+1 in repo) | **P1** | Reconcile count |
| Verify `embedUrl` on all episodes | **P1** | Grep empty embedUrl |
| Platform links (Apple, Spotify, YouTube) | **P2** | Spot-check 5 episodes |

### Practice areas (~12 in repo, 6/9 on sheet)

| Task | Priority | Approach |
|------|----------|----------|
| Build 0–3 missing from manifest | **P1** | Compare Practice Areas tab |
| Sample QA per practice area | **P1** | Visual + meta on all 12 if time, else sample 6 |

---

## Missing service URLs

**Action:** Export the **Service Pages** tab from the migration tracker and paste URLs below. Known built pages are in [PAGE-INVENTORY.md](./PAGE-INVENTORY.md).

| Manifest URL | Status | Notes |
|--------------|--------|-------|
| `/law-firm-seo-services/` | Built (custom) | P0 QA |
| `/law-firm-ppc/` | Built (custom) | P0 QA |
| `/law-firm-web-design/` | Built (custom) | P0 QA |
| `/meta-ads/` | Built (`ServiceTemplate`) | P1 QA |
| `/law-firm-branding/` | Built | P1 QA |
| `/email-marketing-for-law-firms/` | Built | P1 QA |
| `/social-media-for-law-firms/` | Built | P1 QA |
| `/ai-chatbot-for-law-firms/` | Built | P1 QA |
| `/bilingual-seo-law-firms/` | Built | P1 QA |
| _Add rows from sheet_ | Missing? | Build page or 301 redirect |

---

## Phase 4 — QA & sign-off (after editing)

Run on **staging.goconstellation.com** after `main` deploy.

- [ ] **P0** [QA-BREAKDOWN.md](./QA-BREAKDOWN.md) P0 checklist complete
- [ ] **P0** [QA-PROCEDURES.md](./QA-PROCEDURES.md) Phase 4 sign-off recorded
- [ ] **P0** Top 20 URLs meta audit (title, description, canonical)
- [ ] **P0** Header/footer nav—no broken dropdown links
- [ ] **P0** Link crawl on preview or staging; document 404 exceptions
- [ ] **P1** Homepage Backstop or approved manual parity ([VISUAL-QA.md](./VISUAL-QA.md))
- [ ] **P1** Mobile pass: `/`, `/book-call/`, 3 flagship services
- [ ] **P1** Lighthouse spot-check on `/`, `/book-call/`, one blog

---

## Phase 5 — Launch (after migration + QA)

Not started today—track in [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md).

- [ ] **P0** DNS: `www.goconstellation.com` → Astro on Cloudflare
- [ ] **P0** GHL + ClickUp production domain allowlists
- [ ] **P0** Production `robots.txt` allow crawl + submit sitemap in Search Console
- [ ] **P0** Disable staging basic auth on production project (or use separate prod project)
- [ ] **P0** 48h post-cutover monitoring (404s, GSC, booking completions)
- [ ] **P1** Decide WordPress shutdown date after `wp-content` migration complete

---

## Suggested work order (calendar)

| Week | Focus |
|------|--------|
| **1** | Phase 0 scope lock + Phase 1 global fixes (`CoreTemplate`, GTM plan, examples noindex) |
| **2** | Phase 2 P0 pages (homepage, book-call, 3 flagship services, hubs) |
| **3** | Missing service builds + Phase 2 P1 services/partner pages |
| **4** | Phase 3 sampled blog/city/podcast QA + redirect map |
| **5** | Phase 4 sign-off on staging |
| **6** | Phase 5 cutover ([LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md)) |

**Parallel:** Sheet reconciliation can happen in week 1 alongside P0 edits.

---

## Quick commands

```bash
# WP image hotlinks (find pages needing asset migration)
grep -r "wp-content/uploads" src/ --include="*.astro" -l

# Empty podcast embeds
grep -r 'embedUrl=""' src/pages/podcasts/ --include="*.astro" -l

# Production build gate
npm run build && npm run preview
```

---

## Related documentation

| Doc | Use for |
|-----|---------|
| [PAGE-INVENTORY.md](./PAGE-INVENTORY.md) | Templates, standalone pages, category scale |
| [PROGRESS.md](./PROGRESS.md) | Phase status and repo audit |
| [PROGRESS-PERCENTAGE.md](./PROGRESS-PERCENTAGE.md) | Deploy-ready percentages |
| [QA-BREAKDOWN.md](./QA-BREAKDOWN.md) | Full Phase 4 task list |
| [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) | DNS cutover runbook |
| [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) | How to edit blog/city content |
| [TEMPLATES.md](./TEMPLATES.md) | Template props and slots |
