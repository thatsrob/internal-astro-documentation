# Deployment Guide

This guide explains **how the Astro site gets built and published**: local preview, automated staging on Cloudflare Pages, and what to plan for when pointing production traffic at the new stack.

For day-to-day commands, see [DEVELOPMENT.md](./DEVELOPMENT.md). For content and SEO fields during migration, see [CONTENT-GUIDE.md](./CONTENT-GUIDE.md).

---

## Environments at a glance

| Environment | What it is | URL (typical) |
|-------------|------------|----------------|
| **Local dev** | `npm run dev`: hot reload | `http://localhost:4321` |
| **Local production preview** | `npm run build` + `npm run preview` | `http://localhost:4321` (built assets) |
| **Staging (Astro)** | Cloudflare Pages project **`con-staging`** | `*.pages.dev` or custom domain (e.g. `staging.goconstellation.com`) |
| **Production (live)** | WordPress today; Astro after cutover | `https://www.goconstellation.com` |

This repo is named **con-staging** because it targets the **staging** Cloudflare Pages project while the public site may still run on WordPress. Merging to `main` does **not** automatically replace production WordPress until DNS and hosting are switched.

---

## What gets deployed

Astro is configured as a **static site**:

```js
// astro.config.mjs
output: 'static',
site: 'https://www.goconstellation.com',
```

**Build output:** everything in `dist/` after:

```bash
npm run build
```

That folder contains:

- Pre-rendered HTML for every route under `src/pages/`
- Copied `public/` assets (images, fonts, CSS)
- Bundled CSS/JS from Astro/Vite where applicable

There is **no Node server** in production, Cloudflare serves files from the CDN.

---

## Automated deploy (GitHub Actions)

**Workflow file:** `.github/workflows/deploy.yml`

**Trigger:** Push to branch **`main`** only (not pull requests, not other branches).

### Pipeline steps

```
push to main
    → checkout
    → setup Node 22 (npm cache)
    → npm ci
    → npm run build
    → wrangler pages deploy dist --project-name con-staging
```

| Step | Purpose |
|------|---------|
| `npm ci` | Clean install from `package-lock.json` |
| `npm run build` | Must pass; ~400 pages |
| `wrangler pages deploy` | Upload `dist/` to Cloudflare Pages |

### Required GitHub secret

| Secret | Used for |
|--------|----------|
| `CLOUDFLARE_API_TOKEN` | Authenticate Wrangler to Cloudflare |

Create in **GitHub → repo → Settings → Secrets and variables → Actions**.

The token needs permission to deploy to **Cloudflare Pages** for the account that owns project `con-staging`. Use a scoped API token (Pages Edit), not the global account key.

### Cloudflare identifiers in the workflow

The workflow also sets:

```yaml
accountId: 199fb3a85bb95d4bc7b8e966ee418d62
command: pages deploy dist --project-name con-staging
```

- **`con-staging`**: Cloudflare Pages **project name** (staging site).
- **`accountId`**: Cloudflare account ID (public in repo; not secret, but tied to the org).

There is **no `wrangler.toml`** in this repository; the deploy command is fully inline in the workflow.

### What CI does *not* do

- No deploy on PRs (no preview environments in workflow)
- No tests or lint step before build
- No automatic production (`www`) deploy
- No cache purge beyond Cloudflare’s default CDN behavior after deploy

---

## Manual deploy

Use when you need to ship without pushing to `main`, or when debugging Wrangler locally.

### Prerequisites

- Node ≥ 22.12 (match `package.json` `engines`)
- Cloudflare API token with Pages deploy access
- Wrangler (via `npx`, no global install required)

### Commands

```bash
# From repo root
npm ci
npm run build

# Authenticate (one-time per machine)
export CLOUDFLARE_API_TOKEN=your_token_here

# Deploy same artifact as CI
npx wrangler pages deploy dist --project-name con-staging
```

Wrangler prints the deployment URL when finished (Cloudflare `*.pages.dev` URL and/or custom domain if configured in the dashboard).

### Match CI exactly

If staging looks different from what you expected after a local deploy:

1. Confirm you ran `npm run build` (not only `npm run dev`)
2. Confirm project name is `con-staging`
3. Confirm you are on the commit you intend to ship

---

## Local preview before deploy

**Development (unoptimized):**

```bash
npm run dev
```

**Production build (what Cloudflare gets):**

```bash
npm run build
npm run preview
```

Always run **`npm run build`** before merging to `main` if you touched Astro pages, config, or global CSS. CI will fail the same way locally failed.

---

## Branch and release workflow

Recommended flow for the team:

1. Work on a **feature branch**
2. Open PR → review (no auto-deploy from PR in current setup)
3. **`npm run build`** locally or rely on CI after merge
4. **Merge to `main`** → staging deploy runs automatically
5. Verify on staging URL
6. When ready for cutover, follow [Production cutover](#production-cutover) below

There is no separate `staging` git branch required; **`main` is the deploy branch** for Astro staging.

---

## Staging vs production URLs

### `astro.config.mjs` `site`

```js
site: 'https://www.goconstellation.com',
```

Used for Astro’s absolute URL generation (sitemap-related features if enabled, some meta defaults). It does **not** change where Cloudflare hosts the staging project.

### Canonical URLs in page files

Many pages set:

```astro
canonicalUrl="https://goconstellation.com/some-path/"
```

even when you view them on staging. That is intentional for SEO continuity during migration but means:

- View-source on staging may show production canonicals
- Social/debug tools may reference production URLs

See [CONTENT-GUIDE.md](./CONTENT-GUIDE.md#urls-vs-canonicals).

### Redirects

Client-side/build-time redirects in `astro.config.mjs` apply to the **deployed Astro site** (staging once live, production after cutover). They do not change WordPress until traffic hits Astro.

---

## Cloudflare Pages dashboard

Tasks usually done in the **Cloudflare dashboard** (not in this repo):

| Task | Where |
|------|--------|
| Custom domain for staging | Pages → `con-staging` → Custom domains |
| Production domain (after cutover) | Same or separate Pages project |
| Environment variables | Pages project settings (if added later) |
| Rollback to previous deployment | Deployments → view history → rollback |
| Build settings | Not used; build happens in GitHub Actions, not Cloudflare build |

This project uses **GitHub Actions as the builder** and Cloudflare only as the **static host** (`pages deploy` with prebuilt `dist/`).

---

## Production cutover

Cutover is a **hosting/DNS project**, not only a git merge. Use **[LAUNCH-CHECKLIST.md](./LAUNCH-CHECKLIST.md)** for the full phased checklist. Summary:

### Before switch

- [ ] `npm run build` passes on `main`
- [ ] Staging reviewed on real Cloudflare URL (all critical templates, forms, embeds)
- [ ] Redirect map complete (`astro.config.mjs` + any Cloudflare bulk redirects)
- [ ] Canonicals and important meta tags verified
- [ ] `book-call`, support iframe, podcast embeds tested on staging domain ([INTEGRATIONS.md](./INTEGRATIONS.md))
- [ ] Analytics/tag manager plan (if moving from WordPress): GTM not in Astro yet; see [INTEGRATIONS.md](./INTEGRATIONS.md)
- [ ] Sitemap/Search Console plan (`sitemap` still linked to WP in footer today; update when Astro sitemap exists)

### Hosting options

| Approach | Notes |
|----------|--------|
| **Same Pages project** | Point `www.goconstellation.com` at `con-staging` after rename/reconfig |
| **New Pages project** | e.g. `con-production`; update workflow `project-name` and secrets |
| **Cloudflare redirect** | WordPress origin → Astro origin during phased migration |

Coordinate with whoever manages DNS (likely Cloudflare DNS for goconstellation.com).

### After switch

- [ ] Smoke-test top URLs (homepage, `/book-call/`, major service pages, sample blog/city)
- [ ] Monitor 404s in Cloudflare analytics
- [ ] Submit updated sitemap in Search Console
- [ ] Keep WordPress available read-only briefly for rollback if needed

---

## Rollback

### Staging (Cloudflare)

In Cloudflare Pages → **con-staging** → **Deployments**, promote a previous successful deployment. No git revert required for a quick rollback.

### Git

Revert the merge commit on `main` and push; CI will redeploy the older site.

### Production WordPress

Until cutover, production remains WordPress. Rollback from Astro means restoring DNS to the previous origin.

---

## Pre-merge checklist (deploy gate)

- [ ] `npm run build` succeeds locally
- [ ] Spot-check changed routes with `npm run preview`
- [ ] No secrets committed (`.env`, API tokens)
- [ ] Large accidental binaries not added to `public/`
- [ ] If SEO-sensitive: canonicals and redirects intentional for this change

---

## Troubleshooting deploy failures

### CI fails at `npm run build`

- Read the Actions log for the first Astro error (file path + line)
- Fix HTML/syntax in that `.astro` file
- Re-run locally: `npm run build`

### CI fails at Wrangler step

| Symptom | Likely cause |
|---------|----------------|
| Authentication error | Missing or expired `CLOUDFLARE_API_TOKEN` secret |
| Project not found | Wrong project name or token lacks access to account `199fb3a...` |
| Upload failed | Transient Cloudflare error; re-run workflow |

### Deploy succeeded but site looks wrong

- Hard refresh / incognito (CDN cache)
- Confirm you are on the **staging** URL, not WordPress production
- Confirm latest deployment is “Active” in Cloudflare dashboard
- Compare deployment time to merge time

### Staging works, production WordPress unchanged

Expected until DNS/cutover. This repo deploys **con-staging**, not live WordPress.

### Node version mismatch

`package.json` requires `>=22.12.0`. CI uses Node 22. Use the same locally:

```bash
node -v   # should be 22.12+
```

---

## Security notes

- **Never commit** `CLOUDFLARE_API_TOKEN` or put it in `.env` that gets shared
- API token should be **scoped** to minimum Pages permissions
- `accountId` in the workflow is visible in git, which is normal for Cloudflare but confirm it is the correct account
- Rotate tokens if leaked

---

## Related files

| Path | Role |
|------|------|
| `.github/workflows/deploy.yml` | CI/CD pipeline |
| `astro.config.mjs` | Static output, `site`, redirects |
| `package.json` | `build` script, Node engine |
| `dist/` | Build output (gitignored) |
| `.gitignore` | Ignores `dist/`, `.env` |

---

## Related documentation

| Document | Topics |
|----------|--------|
| [DEVELOPMENT.md](./DEVELOPMENT.md) | `dev`, `build`, `preview` |
| [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) | Canonicals, publishing workflow |
| [SITE-OVERVIEW.md](./SITE-OVERVIEW.md) | Architecture summary |
| [MIGRATION-TOOLING.md](./MIGRATION-TOOLING.md) | Pre-deploy migration QA |
