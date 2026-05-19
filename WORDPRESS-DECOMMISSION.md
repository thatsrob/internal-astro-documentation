# WordPress decommission

When and how to turn off the old WordPress site after Astro is live on **goconstellation.com**. This is not a “delete the server on launch day” guide—it’s the sequence that keeps SEO, forms, and images from breaking.

**Do this after:** Phase 4 QA sign-off and DNS cutover ([LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md)).  
**Related:** [DEPLOYMENT.md](./DEPLOYMENT.md) · [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md) · [INTEGRATIONS.md](./INTEGRATIONS.md) · [SEO.md](./SEO.md) · [PERFORMANCE-AND-CSS.md](./PERFORMANCE-AND-CSS.md)

---

## What “decommission” means here

WordPress today is the **live** site: Divi theme, `wp-content/uploads`, Ninja Forms, `sitemap_index.xml`, admin, plugins, probably a separate host or VPS.

Astro on Cloudflare is **staging** until DNS points production traffic at it.

**Decommission** = production no longer depends on WordPress for:

- Public HTML
- Images at `goconstellation.com/wp-content/uploads/...`
- Forms posting to `wp-admin/admin-ajax.php`
- The XML sitemap WordPress generates
- Analytics tags that only exist in the WP theme

You can **keep** WordPress running read-only for a while as a rollback safety net. Full shutdown (cancel hosting, delete files) comes later—often **1–4 weeks** after a stable Astro launch.

---

## Timeline (typical)

```
Before cutover     WordPress = live site     Astro = staging only
Cutover day        DNS → Cloudflare Pages   WordPress still up, not receiving traffic
Days 1–7           Monitor 404s, forms, GSC Astro = live
Days 7–28          Stable?                  WordPress read-only backup
After sign-off     Turn off WP hosting      Astro only
```

Adjust the window with whoever owns the business risk. Don’t delete WordPress before you’re confident in redirects and embeds.

---

## Before you touch DNS (must be done on Astro)

These are WordPress dependencies hiding in the Astro repo. If you skip them, “turning off WP” breaks the new site—not the old one.

### 1. Images still on `wp-content`

Several Astro pages still load images from the live WordPress CDN:

```bash
grep -r "wp-content/uploads" src/ --include="*.astro" -l
```

Migrate those files to `public/images/` (or another CDN you control) and update the paths **before** cutover. Priority: `book-call.astro`, `CityTemplate.astro`, homepage badges, any blog hero that still hotlinks.

See [PERFORMANCE-AND-CSS.md](./PERFORMANCE-AND-CSS.md) and [TODO.md](./TODO.md).

### 2. Redirects for old URLs

WordPress ranked many URLs that Astro serves at a **different path** (blog posts at root vs `/blog/`, case studies at `/case-study-foo/` vs `/case-study/foo/`, etc.).

Only **three** redirect pairs live in `astro.config.mjs` today. The rest need Astro redirects and/or **Cloudflare bulk redirect** rules.

Full map: [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md).

### 3. Integrations moved off WordPress

| Service | On WordPress today | Must work on Astro host |
|---------|-------------------|-------------------------|
| **Book a call** | LeadConnector / GHL embed | `/book-call/` — domain allowlist in GHL |
| **Support** | ClickUp iframe | `/support-center/` |
| **Analytics** | GTM in WP theme | Add `GTM-PDGND8M` to `BaseLayout` ([INTEGRATIONS.md](./INTEGRATIONS.md)) |
| **Forms in content** | Ninja Forms, `admin-ajax` | Remove or replace in migrated HTML—should not fire on Astro pages |

Test on **staging.goconstellation.com** (or your Pages URL), not only localhost.

### 4. SEO files

| Item | WordPress | Astro before cutover |
|------|-----------|----------------------|
| `robots.txt` | Allows indexing | Staging has `Disallow: /` — **replace** for production |
| Sitemap | `/sitemap_index.xml` | Ship `/sitemap.xml` (Astro sitemap or script) |
| Footer sitemap link | Points at WP | Update `SiteFooter.astro` ([NAVIGATION.md](./NAVIGATION.md)) |

### 5. Staging-only stuff

- Remove or relax **basic auth** on the production Pages project when the real domain goes live (`functions/_middleware.ts` is for staging).
- Handle `/examples/*` — noindex or drop from production build.

---

## Cutover day (WordPress stays up)

1. Final `npm run build` on the release commit; deploy to the Pages project that will serve production.
2. Point DNS (or Cloudflare custom domain) at Astro — see [DEPLOYMENT.md](./DEPLOYMENT.md).
3. Smoke-test on the **production domain** ([LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) URL table).
4. **Do not** delete WordPress yet. **Do not** cancel WP hosting yet.

WordPress is your rollback: point DNS back to the old origin if something critical fails.

**Rollback owner** and steps should be written down before the switch (name + phone/Slack, not just “call IT”).

---

## First week on Astro (WordPress idle)

While traffic hits Cloudflare, WordPress sits unused but available.

**Watch:**

- Google Search Console — coverage, crawl errors, sitemap processing
- Cloudflare analytics — 404 spikes
- GHL — bookings still coming in
- ClickUp — support tickets still submitting
- Real-time GTM / GA — traffic on Astro host

**Crawl** top landing pages (GSC export or Screaming Frog) for:

- Broken internal links to old WP paths
- Images still pointing at `wp-content`
- Redirect chains (301 → 301 → 200)

Add missing redirects to `astro.config.mjs` or Cloudflare as you find them. Log them in [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md).

---

## When it’s safe to shut WordPress down

There’s no single automatic signal. You’re looking for a stable period where the list below is true for **at least 1–2 weeks** (many teams use 2–4).

| Check | Done? |
|-------|-------|
| No business-critical 404s on top 50 URLs | |
| Forms and booking work without WP | |
| No Astro pages depend on `wp-content` hotlinks | |
| Analytics trusted on Astro (GTM firing) | |
| Search Console not showing mass “soft 404” or redirect errors | |
| Team agrees rollback to WP is unlikely | |
| Full backup of WordPress taken (files + DB) | |

Get explicit sign-off from whoever owns the site (Patrick / marketing lead)—not only dev.

---

## How to shut it down (order matters)

### 1. Backup everything

Before deleting anything:

- **Database** export (phpMyAdmin, WP-CLI `wp db export`, or host backup)
- **`wp-content/uploads`** — full copy (years of media; Astro may not have every file)
- **Theme/plugin list** — note versions for forensic/debug
- Optional: static export or crawl of any URL **not** in the Astro manifest (old campaigns, landing pages)

Store backups somewhere that **isn’t** the WordPress server you’re about to cancel.

### 2. Stop public access (soft shutdown)

Options (pick what your host allows):

- Put WordPress behind **HTTP auth** or IP allowlist (dev access only)
- Return **410 Gone** or **301 to Astro** at the old WP origin if the hostname still resolves somewhere
- Park the domain on the old host with a single “site moved” page (usually unnecessary if DNS already points to Cloudflare)

Goal: if an old bookmark hits the WP server IP directly, it shouldn’t serve a full duplicate indexed site.

### 3. Disable WordPress cron and forms

- Disable WP cron or stop `wp-cron.php` from being triggered (stops background tasks firing)
- Deactivate form plugins that send email or webhooks (avoid ghost submissions if someone hits old admin URLs)
- Lock **`/wp-admin`** — strong passwords, 2FA offboarding, or remove admin users except break-glass

### 4. Cancel or downsize hosting

- Cancel the VPS / managed WordPress plan **after** backups verify
- Keep DNS at Cloudflare (that’s Astro now—not the old host)
- Release any **staging** WordPress subdomain only if nothing else uses it

### 5. Clean up integrations

- Remove production domain from WordPress-specific tools if duplicated on Astro
- Confirm GHL, ClickUp, GTM only need the Astro URLs going forward
- Update any email signatures, ads, or PDFs that still link to old WP-only paths

---

## What you can remove from day-to-day workflow

After decommission, the team should **not** need:

- wp-admin for content (content is Git + PRs)
- Divi for layout changes on migrated pages
- WordPress plugins for SEO on the main site (Astro `canonicalUrl`, `robots.txt`, sitemap)
- Refore/export pipelines for **new** pages—only for net-new migration batches if any URLs remain

New content workflow: edit `.astro` files or run migration scripts → `main` → CI deploy. See [CONTENT-GUIDE.md](./CONTENT-GUIDE.md).

---

## What to keep archived

| Asset | Why keep it |
|-------|-------------|
| Database dump | Recover old post meta, redirects, form entries |
| `uploads/` folder | Images never migrated; legal/historical reference |
| Redirect plugin export (if used) | Reconcile “why did this URL exist?” |
| Old `sitemap_index.xml` | Compare to Astro sitemap |
| Access logs (if available) | Top URLs that might lack redirects |

You don’t need to keep WordPress **running** to keep these—just the files.

---

## Things that break if you shut WP too early

| Symptom | Likely cause |
|---------|----------------|
| Broken images on Astro | `wp-content` hotlinks still in `src/` |
| 404 on old blog URLs | Missing 301 from root to `/blog/` or vice versa |
| Book-call blank | GHL still allowlisting only WP hostname |
| Analytics flatlines | GTM never added to `BaseLayout` |
| Footer sitemap 404 | Still linking to `sitemap_index.xml` |
| Duplicate site in Google | WP still public and crawlable at old origin |

---

## Relationship to this repo

| System | Role after decommission |
|--------|-------------------------|
| **GitHub `main`** | Source of truth for site content |
| **GitHub Actions** | Build + deploy `dist/` to Cloudflare Pages |
| **Cloudflare Pages** | Hosts static HTML |
| **WordPress** | Off (archived backups only) |

The note in some older docs about a “VPS `.env` build hook” may refer to **WordPress-era** deploy plumbing—not this Astro pipeline. Astro deploys via `.github/workflows/deploy.yml` and Wrangler.

---

## Checklist (printable)

**Pre-cutover (block WP shutdown if incomplete)**

- [ ] No critical `wp-content` dependencies in Astro source
- [ ] Redirect map for top URLs + sheet manifest
- [ ] GTM on Astro
- [ ] Production `robots.txt` + `sitemap.xml`
- [ ] GHL + ClickUp tested on production domain
- [ ] Phase 4 QA sign-off

**Cutover**

- [ ] DNS to Cloudflare Pages
- [ ] Smoke tests pass
- [ ] WordPress left running but not in DNS

**Stable period (1–4 weeks)**

- [ ] GSC + 404 monitoring clean enough for stakeholders
- [ ] Forms and analytics confirmed
- [ ] Written sign-off to decommission WP

**Shutdown**

- [ ] DB + uploads backup stored off-server
- [ ] WP admin locked / host access restricted
- [ ] Hosting cancelled
- [ ] Backups documented (where, who can restore)

---

## Related docs

| Doc | Use for |
|-----|---------|
| [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md) | Full cutover phases |
| [DEPLOYMENT.md](./DEPLOYMENT.md) | DNS, Pages projects, rollback |
| [REDIRECTS-AND-URLS.md](./REDIRECTS-AND-URLS.md) | URL mapping |
| [INTEGRATIONS.md](./INTEGRATIONS.md) | GHL, ClickUp, GTM |
| [SEO.md](./SEO.md) | robots, sitemap, Search Console |
| [TODO.md](./TODO.md) | Pre-launch tasks still open |
