# Integrations Guide

This guide documents **third-party services, embeds, and external dependencies** in the Astro site: what is wired up today, what still lives on WordPress, and what to verify before production cutover.

For deployment and staging access, see [DEPLOYMENT.md](./DEPLOYMENT.md). For the full cutover checklist, see [LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md). For SEO metadata (not tracking), see [SEO.md](./SEO.md).

---

## Overview

The site is **static HTML on Cloudflare Pages**. There is no application server, database, or CMS API in this repo.

| Category | How it connects |
|----------|-----------------|
| **Conversion** | Third-party iframes + scripts on specific pages |
| **Media** | YouTube embeds, outbound links to Spotify/Apple/etc. |
| **Social** | Outbound links in header/footer (no embed widgets) |
| **Analytics** | **Not** in Astro `BaseLayout` today; still on production WordPress |
| **Hosting** | GitHub Actions → Wrangler → Cloudflare Pages |
| **Dev tooling** | Anthropic API, BackstopJS, Refore (migration only) |

Most pages are self-contained. Integrations cluster on **book-call**, **support-center**, **podcasts**, and **legacy migrated HTML/CSS**.

---

## Active visitor-facing integrations

### LeadConnector / GoHighLevel (booking)

**Purpose:** Schedule discovery/strategy calls (primary conversion path).

| Item | Value |
|------|--------|
| **Where** | `src/pages/book-call.astro` |
| **Widget URL** | `https://api.leadconnectorhq.com/widget/booking/7WWVTTJsRrDeHG7AJqpY` |
| **Helper script** | `https://link.msgsndr.com/js/form_embed.js` |
| **Widget ID** | `7WWVTTJsRrDeHG7AJqpY` (in iframe `src` and `id`) |

Implementation:

```astro
<iframe src="https://api.leadconnectorhq.com/widget/booking/7WWVTTJsRrDeHG7AJqpY" ... />
<script src="https://link.msgsndr.com/js/form_embed.js" type="text/javascript"></script>
```

**Template variant:** `BookACallTemplate.astro` supports an optional `calendarUrl` iframe prop, but the example page does **not** pass a URL. It shows a static CTA fallback instead. Production booking lives on the custom `book-call.astro` page above.

**Cutover checks:**

- [ ] Embed loads on staging domain (some widgets restrict allowed origins)
- [ ] Height/scrolling acceptable on mobile
- [ ] Confirm widget ID still matches the live GHL calendar
- [ ] Thank-you / confirmation URL still correct in GHL settings (`/thank-you/` on site)

---

### ClickUp (support tickets)

**Purpose:** Client support ticket intake.

| Item | Value |
|------|--------|
| **Where** | `src/pages/support-center.astro` |
| **Form URL** | `https://forms.clickup.com/2368165/f/288n5-187387/J9I6S8M79VHI11E0MS` |

Embedded as a full-width iframe (~700px tall). No ClickUp SDK in the repo; only the hosted form URL.

**Cutover checks:**

- [ ] Form submits successfully from Astro host
- [ ] Notifications/routing unchanged in ClickUp workspace `2368165`

---

### YouTube (podcasts and some blog posts)

**Purpose:** Episode video playback.

| Item | Value |
|------|--------|
| **Template** | `PodcastTemplate.astro`: `embedUrl` prop → `<iframe src={embedUrl}>` |
| **Blog** | Some posts pass `embedUrl` to `BlogPostTemplate` (e.g. `how-we-approach-on-page-optimization.astro`) |
| **Format** | `https://www.youtube.com/embed/VIDEO_ID` |

`PodcastTemplate` also links out to platform URLs when set:

| Prop | Example use |
|------|-------------|
| `youtubeUrl` | Watch on YouTube |
| `spotifyUrl` | Constellation show on Spotify |
| `appleUrl` | Apple Podcasts |
| `audibleUrl` | Audible |
| `shortVideos` | Related Shorts/clips |

Podcast index (`src/pages/podcasts/index.astro`) links to the main Spotify show: `https://open.spotify.com/show/12k368acxfInHuPuft22yE`.

**Privacy / performance:** YouTube iframes load third-party cookies/scripts when the iframe is in view. No cookie-consent banner exists in Astro today.

---

### Social profiles (links only)

**Purpose:** Brand presence; no social widgets or share SDKs in Astro components.

| Platform | URL (footer / thank-you) |
|----------|---------------------------|
| Facebook | `https://facebook.com/constellationmktg` |
| Instagram | `https://www.instagram.com/constellationmktg/` |
| LinkedIn | `https://www.linkedin.com/company/constellation-marketing` |
| X | `https://x.com/GoConstellation` |
| YouTube | `https://www.youtube.com/@constellationmktg` |

Defined in `SiteFooter.astro` and duplicated on `thank-you.astro`.

**Legacy note:** Some migrated blog HTML still contains **Divi Monarch** social-share buttons pointing at `facebook.com/sharer.php` with production URLs. That is editorial markup, not a global integration.

---

## Not integrated in Astro (gaps vs WordPress)

### Google Tag Manager

Production WordPress exports reference **GTM container `GTM-PDGND8M`** (e.g. `sections-output/section-13.html`, legacy section HTML).

**`BaseLayout.astro` does not include GTM** (or GA4, Meta Pixel, etc.). Staging Astro builds send **no sitewide analytics tags** unless you add them.

**Cutover task:** Port the GTM snippet (or replacement tags) into `BaseLayout` or a small `Analytics.astro` component, and verify tags in GTM preview mode on the Astro host.

---

### HubSpot forms

**Not embedded** in current Astro templates.

Legacy CSS still references HubSpot:

- `divi-theme-options.css`: `#hsForm_...` overrides
- Migrated pages may mention HubSpot in **content** only

Footer/contact flows on WordPress used **Ninja Forms** (see below), not HubSpot embeds on the live homepage in this rebuild.

---

### Ninja Forms (WordPress)

**Not functional** on Astro pages.

Evidence in repo:

- Ninja Forms styles in `public/css/production-extracted.css` and `public/styles/production.css`
- Form markup/scripts in `sections-*/` migration artifacts pointing at `wp-admin/admin-ajax.php`

Those forms **submitted to WordPress**. Without WP, any leftover Ninja markup in unmigrated HTML will not submit.

**Action:** Ensure all lead capture routes to **book-call** (GHL) or **support-center** (ClickUp), not legacy Ninja shortcodes.

---

### Live chat

No chat widget (Intercom, HubSpot chat, Client Chat Live, etc.) in `BaseLayout` or shared components. Blog posts *review* chat products; they do not load them.

---

## External asset dependencies

### WordPress media CDN

Many pages still reference images on production WordPress:

```
https://goconstellation.com/wp-content/uploads/...
```

Examples:

- Testimonial photos on `book-call.astro`
- Badge images on `HomepageTemplate-v2.astro`
- Case study cards in `CityTemplate.astro`

**Risk:** If WordPress is decommissioned before images are mirrored to `public/images/`, those URLs **404**.

**Action:** Migrate hot-path images to `public/` and update `src` attributes, or proxy/cache them on Cloudflare.

### Self-hosted fonts

Headings use **self-hosted Poppins** (`public/fonts/`, declared in `src/styles/global.css`). No Google Fonts runtime request.

`@fontsource/poppins` is in `package.json` but **not imported**, safe to ignore or remove later.

---

## Infrastructure integrations

### Cloudflare Pages (hosting)

| Item | Detail |
|------|--------|
| **Project** | `con-staging` |
| **Deploy** | GitHub Actions → `wrangler pages deploy dist --project-name con-staging` |
| **Build** | Static `dist/` from `npm run build` (no Cloudflare build step) |

See [DEPLOYMENT.md](./DEPLOYMENT.md).

---

### Staging basic auth (Cloudflare Functions)

**File:** `functions/_middleware.ts`

Protects the **Cloudflare Pages** deployment with HTTP Basic Auth when env vars are set:

| Variable | Purpose |
|----------|---------|
| `BASIC_AUTH_USER` | Username |
| `BASIC_AUTH_PASS` | Password |

If either is missing, **all requests return 401** (fail closed).

Configure in **Cloudflare Pages → con-staging → Settings → Environment variables** (not in this git repo).

**Note:** Local `npm run dev` / `npm run preview` does **not** run this middleware; only the Pages deployment does.

---

### GitHub Actions

**Workflow:** `.github/workflows/deploy.yml`

| Secret | Purpose |
|--------|---------|
| `CLOUDFLARE_API_TOKEN` | Wrangler deploy to Pages |

No other third-party secrets in CI today.

---

## Development-only integrations

These run on developer machines or in optional scripts, not in the static site visitors receive.

### Anthropic (Claude)

| Item | Detail |
|------|--------|
| **Package** | `@anthropic-ai/sdk` in `package.json` |
| **Usage** | `scripts/fix-loop.mjs`: reads Backstop diff images, calls Claude API, edits `HomepageTemplate.astro` |
| **Auth** | `ANTHROPIC_API_KEY` in env or `.env` (gitignored) |

Never expose this key in the browser or commit it to the repo.

---

### BackstopJS

Visual regression vs production homepage. No external API; local screenshots only. See [VISUAL-QA.md](./VISUAL-QA.md).

---

### Refore

WordPress → static HTML export tool used during homepage migration. See [REFORE-APPROACH.md](../REFORE-APPROACH.md) and [MIGRATION-TOOLING.md](./MIGRATION-TOOLING.md).

---

## Integration map by page type

| Page / area | Integration |
|-------------|-------------|
| Most marketing pages | None (static HTML + local assets) |
| `book-call.astro` | LeadConnector iframe + msgsndr script |
| `support-center.astro` | ClickUp form iframe |
| `thank-you.astro` | None (post-conversion content + social links) |
| `podcasts/*` | YouTube iframe + optional Spotify/Apple/Audible links |
| Some `blog/*` | Optional YouTube `embedUrl` |
| `become-a-partner`, `referral-program` | No embed; CTAs link elsewhere |
| Migrated blog HTML | May contain dead WP share buttons, external links only |

---

## Adding a new integration

Prefer this order:

1. **Iframe or official embed snippet** from the vendor (booking, forms, maps) keeps logic off your build.
2. **Shared Astro component** (e.g. `src/components/Analytics.astro`) included from `BaseLayout` for sitewide tags.
3. **`slot="head"`** on a single page for page-specific scripts.

Avoid:

- Hardcoding API secrets in `.astro` files (they ship to the browser).
- Loading heavy scripts on every page when only one route needs them.

For sitewide GTM:

```astro
<!-- Example pattern: add via component, not copy-paste blindly -->
<script is:inline>
  (function(w,d,s,l,i){...})(window,document,'script','dataLayer','GTM-XXXX');
</script>
```

Use Astro’s guidance for third-party scripts (`is:inline`, defer, Partytown) if performance becomes an issue.

---

## Pre-launch integration checklist

### Must work on staging host

- [ ] `/book-call/`: GHL calendar loads and books a test slot
- [ ] `/support-center/`: ClickUp form submits
- [ ] Sample podcast: YouTube plays
- [ ] Header/footer social links open correct profiles

### Must plan before WordPress shutdown

- [ ] GTM / analytics added to `BaseLayout` (or tag manager container updated for new domain)
- [ ] Critical `wp-content/uploads` images copied to `public/`
- [ ] No live Ninja Forms or `admin-ajax.php` dependencies on indexable pages
- [ ] GHL + ClickUp allowed domains include production hostname
- [ ] Cookie/consent policy if required for EU traffic (not implemented today)

### Staging-only

- [ ] `BASIC_AUTH_*` set on Cloudflare Pages (or removed when staging should be public)
- [ ] `robots.txt` still blocks crawlers on staging if needed ([SEO.md](./SEO.md))

---

## Troubleshooting

| Symptom | Likely cause |
|---------|----------------|
| Blank booking iframe | Widget domain allowlist; ad blocker; wrong widget ID |
| ClickUp form empty | iframe blocked; wrong form URL; ClickUp outage |
| YouTube “Video unavailable” | Bad `embedUrl`; privacy/region restrictions |
| Analytics flat after cutover | GTM never added to Astro; container still firing only on WP |
| Broken testimonial images | `wp-content` URLs after WP takedown |
| 401 on staging URL | Basic auth env vars missing or wrong credentials |

---

## Related documentation

| Document | Topics |
|----------|--------|
| [DEPLOYMENT.md](./DEPLOYMENT.md) | CI/CD, Cloudflare, cutover |
| [SEO.md](./SEO.md) | Meta tags, robots, sitemaps |
| [SITE-OVERVIEW.md](./SITE-OVERVIEW.md) | Architecture, no backend |
| [MIGRATION-TOOLING.md](./MIGRATION-TOOLING.md) | Scripts, Refore, fix-loop |
| [TEMPLATES.md](./TEMPLATES.md) | `PodcastTemplate`, `BookACallTemplate` props |
| [STYLING.md](./STYLING.md) | Legacy HubSpot/Ninja CSS |
