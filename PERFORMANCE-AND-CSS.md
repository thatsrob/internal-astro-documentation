# Performance and CSS

How fast the site feels, why the CSS bundle is so big, and what’s worth fixing before launch vs after. For day-to-day “which file do I edit to change a color,” use [STYLING.md](./STYLING.md)—this doc is about weight, load order, and speed.

**Related:** [STYLING.md](./STYLING.md) · [DEPLOYMENT.md](./DEPLOYMENT.md) · [QA-PROCEDURES.md](./QA-PROCEDURES.md) · [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md)

---

## The honest summary

The Astro build is **fast** (~400 pages in under ten seconds). Page HTML is static and small. What hurts performance on the **browser** side is mostly:

1. **`production-extracted.css`** — about **488KB** on **every** page, whether or not that page uses Divi markup.
2. **Extra CSS per template** — large `<style>` blocks on homepage, flagships, and some templates (copied from WordPress).
3. **Images** — hero photos, case study assets, blog images (WebP helps where migration scripts ran; many posts still hotlink WordPress).
4. **Third-party embeds** — GHL booking iframe, ClickUp, YouTube on podcasts (not under your CSS control).

There is **no PurgeCSS pipeline** in the repo today. Nobody has trimmed the Divi extract yet. That’s a planned “CSS diet,” not a launch blocker unless Lighthouse on money pages looks bad.

---

## What loads on every page

`BaseLayout.astro` wires this in for the whole site:

| Asset | Size (approx.) | Notes |
|-------|----------------|--------|
| `global.css` (via Astro) | ~12KB bundled with Tailwind + theme | Fonts, tokens, base rules |
| `production-extracted.css` | **~488KB** | Static file in `public/css/` — full Divi snapshot |
| Poppins preload | One woff2 | `poppins-700.woff2` in `<head>` |

So a minimal page like `/privacy-policy/` still downloads the same half-megabyte Divi CSS as the homepage. That’s a migration tradeoff: one file keeps `et_pb_*` classes working everywhere without maintaining nine separate 400KB extracts.

### Sitting in `public/css/` but not loaded globally

These were pulled from production by page type. They’re **reference only** unless you manually `<link>` them:

| File | ~Size |
|------|-------|
| `motion-extracted.css` | — |
| `divi-homepage.css` | 452KB |
| `motion-extracted` n/a |
| `motion-extracted` |
| `divi-service.css` | 455KB |
| `motion-extracted` |
| `divi-practice-area.css` | 424KB |
| `divi-city.css` | 416KB |
| `motion-extracted` |
| `divi-blog-post.css` | 411KB |
| `divi-podcast.css` | 437KB |
| `divi-book-a-call.css` | 413KB |
| `divi-framework.css` | 452KB |
| `divi-main-modules.min.css` | 236KB |

Don’t link all of these on top of `production-extracted.css` unless you enjoy duplicate rules and a multi-megabyte first visit.

---

## Why the Divi file is global

WordPress shipped layout through Divi classes (`et_pb_section`, `et_pb_row`, etc.). Hundreds of migrated pages still depend on those class names. The extract was parked in `public/` so Astro/Vite wouldn’t try to process a giant legacy stylesheet on every build ([REFORE-APPROACH.md](../REFORE-APPROACH.md)).

**Good:** Blog HTML and city sections often “just work.”  
**Bad:** Every visitor pays the download cost even on a page that’s 90% custom Astro + inline styles.

---

## Layers and performance (how they stack)

```
Browser request
    │
    ├─ HTML (static, usually fine)
    │
    ├─ production-extracted.css     ← biggest fixed cost
    ├─ Astro bundle (global.css + scoped component CSS)
    ├─ <style slot="head"> per page  ← can be large on homepage / book-call
    └─ inline style="" on elements   ← lots on migrated pages; hard to dedupe
```

**Specificity fights** don’t slow the network much, but they slow **you** down when tuning LCP. Check DevTools → Performance → which stylesheet wins before adding more overrides.

---

## Build and deploy performance

| Metric | Typical value |
|--------|----------------|
| Pages built | ~396 |
| `npm run build` time | ~5–8 seconds locally |
| Output | Static HTML in `dist/` |
| CI | GitHub Actions on push to `main` → Cloudflare Pages |

Build time is not the problem. Cloudflare serves gzip/brotli on CSS automatically, so **488KB uncompressed** might land around ~60–80KB transferred—still non-trivial on mobile, still parse work for the browser.

---

## Fonts

Headings use **self-hosted Poppins** (`public/fonts/poppins-*.woff2`). `BaseLayout` preloads the 700 weight.

`@fontsource/poppins` is in `package.json` but **not imported**—the site uses the woff2 files only. Don’t add a second Poppins load from npm without removing the manual `@font-face` rules.

Body text is mostly system UI stack in `global.css`. Blog and practice templates often set Poppins on `.article-body` anyway, so more font bytes may load when those weights are used.

---

## Images

Performance wins already in the repo:

- City migration script converts heroes to **WebP** under `public/images/`.
- Many pages use local `/images/...` paths.

Still open:

- **`wp-content/uploads` hotlinks** in `src/` — browser hits live WordPress for images on those pages; extra DNS, no control after WP shutdown. Grep: `grep -r "wp-content/uploads" src/ --include="*.astro"`.
- Blog posts with huge inline images from migration.
- No sitewide lazy-load policy beyond whatever migrated HTML already has (`loading="lazy"` in some templates).

---

## Third-party weight (not CSS, but same tab)

| Embed | Pages | Note |
|-------|-------|------|
| LeadConnector / GHL | `/book-call/` | iframe + scripts |
| ClickUp | `/support-center/` | iframe |
| YouTube | Podcasts, some blogs | iframe |
| GTM | **Not in Astro yet** | Will add JS when added to `BaseLayout` |

QA these on **staging hostname**—they don’t behave the same on localhost.

---

## What to measure

No performance budget is enforced in CI. Before launch, manually run **Lighthouse** (Chrome DevTools) on a few URLs:

| URL | Why |
|-----|-----|
| `/` | Heaviest custom layout |
| `/book-call/` | Conversion + iframe |
| One blog post | Long article body + images |
| One city page | `CityTemplate` + images |

Note **LCP** (usually hero or font), **CLS** (images, embeds, fonts), and **Total Blocking Time** (scripts). Save scores in a spreadsheet if you’re tracking regressions.

Targets aren’t written in stone for this project yet. Practical bar: money pages shouldn’t feel obviously worse than production WordPress on the same connection. If Astro is dramatically worse, fix the worst offender first (usually LCP image or CSS blocking render).

---

## CSS diet options (not implemented)

These are the realistic paths when someone has time. Pick one strategy; don’t do all at once without testing.

### Option A — Purge / trim `production-extracted.css`

Run PurgeCSS (or similar) against built HTML in `dist/`, using safelist for `et_pb_*` and dynamic classes. Goal: drop unused Divi rules while keeping migrated pages intact.

**Risk:** Over-aggressive purge breaks a city page or blog callout that uses a rare Divi class. Needs visual QA on samples.

### Option B — Per-template CSS instead of one global file

Stop linking `production-extracted.css` in `BaseLayout`. Link smaller extracts only where needed (`divi-blog-post.css` on blogs, etc.).

**Risk:** Maintenance nightmare; easy to miss a template. Might save bytes on “simple” pages only.

### Option C — Gradual strip Divi classes from markup

Replace Divi HTML with Astro components + Tailwind/`global.css` over time. Shrink the extract as classes disappear.

**Risk:** Slow; correct for long-term health, not a pre-launch weekend.

### Option D — Leave it for v2

Ship with the 488KB file if Lighthouse is acceptable. Track “CSS optimization” as post-launch. Many teams do this when launch date is fixed.

**Current team stance (from progress docs):** PurgeCSS is **recommended before or after cutover**, not a hard gate unless scores fail.

---

## Quick wins (low effort)

| Change | Impact |
|--------|--------|
| Migrate off `wp-content` image URLs | Fewer external requests; survives WP shutdown |
| Don’t add new global `<link>` to unused `motion-extracted` divi-*.css files | Avoid accidental double CSS |
| Compress/resize new images before `public/images/` | LCP on pages you touch |
| Preload only the font weights you actually use | Small head savings |
| Add `width`/`height` on key images | CLS |
| Defer non-critical scripts when GTM lands | TBT |

---

## What not to do

- Edit `production-extracted.css` for a one-off blog typo (huge diff, affects everyone).
- Link multiple full Divi extracts on one page.
- Assume Tailwind purges Divi classes in migrated HTML (it doesn’t scan arbitrary class strings in scraped bodies well).
- Block launch on perfect Lighthouse 100 if production wasn’t 100 either—match business expectations.

---

## Homepage and flagship pages

`HomepageTemplate`, `law-firm-seo-services`, `law-firm-ppc`, `law-firm-web-design`, and `book-call` carry **large inline style blocks** or page-scoped CSS in addition to the global extract. They’re the right place to look if the homepage feels heavy even after CSS diet.

`HomepageTemplate-v2` / `v3` exist for experiments; production should stay on v1 unless you’ve checked perf and links.

---

## Caching after launch

Cloudflare Pages serves static assets with CDN caching. CSS files in `public/css/` get cache headers from Cloudflare. When you **replace** `production-extracted.css`, bump cache (filename hash or query string) or purgers may serve old CSS—worth a note in the deploy runbook.

---

## Checklist

**Before launch (spot-check):**

- [ ] Lighthouse on `/`, `/book-call/`, one blog
- [ ] Mobile throttling on book-call (iframe + CSS)
- [ ] No new 500KB CSS files linked in `BaseLayout`
- [ ] P0 images not on WordPress CDN

**After launch (optional):**

- [ ] PurgeCSS or scoped extract experiment on staging
- [ ] Compare transferred CSS size before/after
- [ ] Re-run Lighthouse on same URLs
- [ ] Document decision in this file

---

## File reference

| Path | Performance role |
|------|------------------|
| `src/layouts/BaseLayout.astro` | Global CSS links, font preload |
| `public/css/production-extracted.css` | Main weight |
| `public/css/divi-*.css` | Unused unless linked |
| `src/styles/global.css` | Small; Tailwind + fonts |
| `src/styles/divi-theme-options.css` | WP custom CSS snapshot |
| `src/layouts/templates/*.astro` | Page-scoped CSS weight |

---

## Related docs

| Doc | Use for |
|-----|---------|
| [STYLING.md](./STYLING.md) | How to edit styles without breaking layout |
| [QA-PROCEDURES.md](./QA-PROCEDURES.md) | Lighthouse in QA phase |
| [PROGRESS-PERCENTAGE.md](./PROGRESS-PERCENTAGE.md) | Performance % in launch readiness |
| [INTEGRATIONS.md](./INTEGRATIONS.md) | Embed and GTM impact |
