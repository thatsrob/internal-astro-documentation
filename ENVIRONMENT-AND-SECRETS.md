# Environment and secrets

Where configuration and credentials live for the Constellation Astro site—what goes in Git, what stays in password managers, and what you need for local dev, CI, staging, and launch.

Nothing in this doc should be copied into Slack or email as plain text. Rotate anything that was ever committed by mistake.

---

## The big picture

This project is **static**: build on GitHub Actions, host on Cloudflare Pages. There is no app database or server-side API in the repo.

Secrets fall into a few buckets:

| Bucket | Examples | Stored where |
|--------|----------|--------------|
| **Deploy** | Cloudflare API token | GitHub Actions secret |
| **Staging access** | HTTP basic auth user/password | Cloudflare Pages env vars |
| **Local dev tools** | Anthropic API key (optional) | Your machine `.env` (gitignored) |
| **Third-party dashboards** | GHL calendar, ClickUp form | Vendor UIs—not in this repo |
| **Public IDs** | GTM container ID, widget IDs in source | Committed in `.astro` files (not secret) |

---

## What must never be committed

- `.env` files with API keys
- Cloudflare global API keys (use scoped tokens instead)
- Staging basic-auth passwords in source code
- WordPress admin credentials
- Private keys, service account JSON, or client secrets for ads/analytics

The repo `.gitignore` should already exclude `.env`. If a secret was committed, rotate it and remove it from history per your org’s security process.

---

## Local development

### Node and commands

- **Node.js ≥ 22.12** (see `package.json` engines).
- No `.env` is required for normal work: `npm install`, `npm run dev`, `npm run build`, `npm run preview`.

### Optional: `.env` at repo root

Used only for **migration/QA tooling**, not for the site build.

| Variable | Required? | Used by |
|----------|-----------|---------|
| `ANTHROPIC_API_KEY` | Only if you run the homepage fix loop | `scripts/fix-loop.mjs` |

Create a file named `.env` in the project root:

```
ANTHROPIC_API_KEY=sk-ant-...
```

The fix-loop script may also read the key from the shell: `ANTHROPIC_API_KEY=sk-... node scripts/fix-loop.mjs`.

**Do not** put `CLOUDFLARE_API_TOKEN` in local `.env` unless you deploy manually with Wrangler—and still do not commit that file.

### Local dev vs staging middleware

`functions/_middleware.ts` enforces **HTTP Basic Auth** on the Cloudflare deployment when `BASIC_AUTH_USER` and `BASIC_AUTH_PASS` are set in **Pages** environment variables.

Running `npm run dev` on localhost **does not** run that middleware. Staging feels “open” locally; production-like auth only applies on the deployed staging URL.

---

## GitHub Actions (deploy to staging)

**Workflow:** `.github/workflows/deploy.yml`  
**Trigger:** Push to `main` only.

| Secret name | Purpose |
|-------------|---------|
| `CLOUDFLARE_API_TOKEN` | Lets Wrangler upload `dist/` to Cloudflare Pages project `con-staging` |

**Where to set it:** GitHub → repository → Settings → Secrets and variables → Actions → New repository secret.

**Token scope:** Create a **Cloudflare API token** with permission to deploy to **Cloudflare Pages** for the account that owns project `con-staging`. Prefer a scoped “Pages Edit” style token, not the account Global API Key.

**Not secret (in workflow YAML):**

- `accountId: 199fb3a85bb95d4bc7b8e966ee418d62`
- `project-name: con-staging`

Those identify the hosting target; they are not credentials.

### What CI does not use

- No `ANTHROPIC_API_KEY` in CI (fix-loop is local only)
- No staging basic-auth secrets in GitHub (those live on Cloudflare Pages)
- No PR preview deploys in the current workflow

---

## Cloudflare Pages (staging site)

Project name: **`con-staging`**.

Build happens in **GitHub Actions**, not in Cloudflare’s build UI. Cloudflare only receives the prebuilt `dist/` folder.

### Staging HTTP Basic Auth

**File:** `functions/_middleware.ts`

| Pages env var | Purpose |
|---------------|---------|
| `BASIC_AUTH_USER` | Username for staging visitors |
| `BASIC_AUTH_PASS` | Password for staging visitors |

**Where to set:** Cloudflare dashboard → Workers & Pages → **con-staging** → Settings → Environment variables (production environment for the Pages project).

If either variable is missing, the middleware returns **401 Unauthorized for everyone**—by design, so an unconfigured deploy is not public.

**Who needs credentials:** Anyone QA’ing `staging.goconstellation.com` (or the `*.pages.dev` URL). Store username/password in the team password manager, not in the repo.

### Removing auth for production

When the real domain points at Astro for public launch, disable or remove basic auth on the **production** Pages project (or use a separate production project without middleware). Plan that in the launch checklist—not something to flip accidentally on the only deploy.

### Other Cloudflare settings (dashboard, not git)

Typical tasks done in the UI:

- Custom domain (e.g. `staging.goconstellation.com`)
- Rollback to a previous deployment
- Bulk redirect rules (launch redirect map)
- DNS for `goconstellation.com` at cutover

---

## Manual deploy with Wrangler

If you deploy without pushing to `main`:

```bash
npm ci
npm run build
CLOUDFLARE_API_TOKEN=your_token npx wrangler pages deploy dist --project-name con-staging
```

Use the same scoped token as GitHub Actions. There is no `wrangler.toml` in the repo; the project name is passed on the command line.

---

## Third-party services (not repo secrets)

These integrations use **public widget URLs or IDs** in source code. The “secret” is usually **domain allowlisting** in the vendor dashboard.

### LeadConnector / GoHighLevel (book a call)

- **Page:** `src/pages/book-call.astro`
- **Widget ID** in iframe URL: `7WWVTTJsRrDeHG7AJqpY`
- **Cutover:** In GHL, allow the Astro hostnames (staging and production) so the embed loads.

No API key in the repo for the booking iframe.

### ClickUp (support center)

- **Page:** `src/pages/support-center.astro`
- **Form URL** is a public hosted form link.

Verify submissions from the staging domain before launch.

### Google Tag Manager

- **Container:** `GTM-PDGND8M` (documented for launch; not yet in `BaseLayout.astro`)
- Container ID is not a secret; it will appear in HTML when added.

Configure tags in the GTM UI. Test with GTM Preview on staging after the snippet is in `BaseLayout`.

### YouTube, Spotify, social links

Outbound or embed URLs only—no credentials in repo.

---

## Migration scripts (local only)

| Script | Credentials |
|--------|-------------|
| `build_blog_posts.py` | None; fetches public production HTML (SSL verify disabled in script—run only against trusted URLs) |
| `build_city_pages.py` | None; same |
| `build_astro_post.py` | Reads local JSON file |
| `fix-loop.mjs` | `ANTHROPIC_API_KEY` |
| Backstop / Playwright | No keys; local dev server on port 4321 |

Update hardcoded **file paths** inside Python scripts (`/Users/patrickcarver/...`) on your machine—that is configuration, not a secret.

---

## Production cutover (who holds what)

Fill in names in your team’s launch doc. Typical ownership:

| Item | Owner role | Where it lives |
|------|------------|----------------|
| Cloudflare account / DNS | Dev or infra | Cloudflare dashboard |
| `CLOUDFLARE_API_TOKEN` (GitHub) | Dev | GitHub secrets |
| Staging basic auth | PM or dev | Password manager + Cloudflare env |
| GHL calendar / domain allowlist | Marketing ops | GoHighLevel |
| ClickUp form workspace | Support ops | ClickUp |
| GTM / GA4 | Marketing | Google Tag Manager / Analytics |
| Search Console | SEO | Google |
| WordPress admin (until decommission) | Whoever owns legacy site | WP host |

---

## Checklist before sharing access with a new teammate

- [ ] Node 22+ installed
- [ ] Repo cloned; `npm install` works
- [ ] Staging URL + basic auth credentials (if QA on staging)
- [ ] No need for Cloudflare token unless they deploy manually
- [ ] Optional: `.env` with `ANTHROPIC_API_KEY` only if they run fix-loop
- [ ] Point them at integrations doc for GHL/ClickUp verification steps

---

## If something leaks

1. **Rotate** the credential immediately (Cloudflare token, staging password, Anthropic key).
2. **Revoke** the old token in the provider UI.
3. **Update** GitHub secrets or Cloudflare env vars with the new value.
4. **Redeploy** if staging auth or deploy broke after rotation.
5. Do not commit the replacement in source control.
