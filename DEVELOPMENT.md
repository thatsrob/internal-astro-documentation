# Development Guide

This guide is for anyone who needs to run, change, or ship the Constellation Marketing site locally. It assumes you have read [SITE-OVERVIEW.md](./SITE-OVERVIEW.md) for the big picture and optionally [ASTRO.md](./ASTRO.md) for how Astro works here; this document goes deeper into **how to work in the repo day to day**.

---

## What you are working on

You are not building a traditional web app. There is no database, no login system, and no API server in this repo. When you run `npm run dev`, Astro starts a local server that compiles `.astro` files into HTML on the fly. When you run `npm run build`, Astro pre-renders every page into static files inside `dist/`, and Cloudflare Pages serves those files as-is.

That means most of your work falls into one of these buckets:

1. **Edit content**: change text, images, or HTML inside a page file.
2. **Edit layout**: change a shared template (header, blog shell, city page layout).
3. **Edit styling**: Tailwind tokens, legacy Divi CSS, or inline styles copied from WordPress.
4. **Run migration tooling**: optional Python/Node scripts that scrape production and generate pages (usually one-off or bulk work).

If something “doesn’t work” locally, ask first: is it a **build** problem, a **styling** problem, or a **third-party embed** (booking widget, podcast player) that only works on the live domain?

---

## Prerequisites

| Requirement | Why |
|-------------|-----|
| **Node.js 22.12 or newer** | Enforced in `package.json` (`engines.node`) |
| **npm** | Dependencies and scripts use npm (lockfile is `package-lock.json`) |
| **Git** | Content and assets live in the repo |

Optional, depending on what you do:

| Tool | When you need it |
|------|------------------|
| **Python 3** | Running `scripts/build_blog_posts.py`, `build_city_pages.py`, etc. |
| **Pillow (`pip install Pillow`)** | `build_city_pages.py` converts images to WebP |
| **Playwright (via BackstopJS)** | Visual regression tests (`npx backstop test`) |
| **Anthropic API key** | Only for `scripts/fix-loop.mjs` (automated homepage diff fixing) |

You do **not** need WordPress, PHP, or a local database.

---

## First-time setup

From the repository root:

```bash
# Use Node 22+ (nvm example)
nvm install 22
nvm use 22

# Install dependencies
npm install

# Start the dev server
npm run dev
```

Open **http://localhost:4321** in your browser. You should see the homepage (routed from `src/pages/index.astro`).

If `npm install` warns about engine version, upgrade Node before continuing, so odd build errors often trace back to an older Node.

---

## Commands you will use every day

| Command | What it does |
|---------|----------------|
| `npm run dev` | Dev server with hot reload at `http://localhost:4321` |
| `npm run build` | Production build → writes static site to `dist/` |
| `npm run preview` | Serves `dist/` locally (use after `build` to catch production-only issues) |
| `npm run astro -- --help` | Astro CLI help (integrations, diagnostics) |

### Dev vs preview: when to use which

- **`npm run dev`**: Normal editing. Fast feedback. Good for content and layout changes.
- **`npm run build` + `npm run preview`**: Closer to what Cloudflare serves. Use this before opening a PR if you changed global CSS, many pages, or anything that might behave differently in production mode.

CI runs `npm run build` only (not `dev`), so a green local `build` is the bar for deploy.

---

## How Astro is configured here

The important settings live in `astro.config.mjs`:

```js
output: 'static',
site: 'https://www.goconstellation.com',
```

**Static output** means every URL is a real HTML file at build time. No server-side rendering, no Astro API routes, no middleware.

**`site`** is used for absolute URLs in some Astro features. Many pages still hardcode `canonicalUrl="https://goconstellation.com/..."` in their frontmatter; when editing SEO, check both the config and the individual page.

**Redirects**: A small set of legacy WordPress URLs are redirected in config (e.g. long mass-tort slug → shorter slug). Add new redirects here when production URLs change, not only in Cloudflare.

**Tailwind**: Wired through the Vite plugin (`@tailwindcss/vite`). Tailwind is imported in `src/styles/global.css`; that file is pulled in by `BaseLayout.astro`.

---

## Repository layout (developer view)

```
con-staging/
├── src/
│   ├── pages/              ← Routes: one file ≈ one URL
│   ├── layouts/
│   │   ├── BaseLayout.astro
│   │   └── templates/      ← Page-type shells (blog, city, homepage, …)
│   ├── components/         ← SiteHeader, SiteFooter only
│   └── styles/
│       ├── global.css      ← Tailwind + fonts + theme tokens
│       └── divi-theme-options.css
├── public/                 ← Copied as-is to site root (/images, /css, /fonts)
├── scripts/                ← Migration & QA (not part of runtime)
├── documents/              ← You are here
├── backstop_data/          ← Screenshot test artifacts
├── astro.config.mjs
└── package.json
```

### Mental model: three layers per page

Almost every public page is built like this:

```
src/pages/some-page.astro     →  thin file: SEO props + pick a template
        ↓
src/layouts/templates/*.astro →  layout, sections, optional <slot> for body HTML
        ↓
BaseLayout + SiteHeader + SiteFooter
```

**Example:** `src/pages/index.astro` only imports `HomepageTemplate` and passes `seoTitle`, `seoDescription`, and `canonicalUrl`. The homepage’s real markup lives in `HomepageTemplate.astro` (~1,500 lines).

**Blog posts** are different only in that the “body” is often hundreds of lines of HTML **inside** the page file, passed into `BlogPostTemplate` as default slot content.

---

## File-based routing (how URLs are chosen)

Astro maps the filesystem under `src/pages/` to URLs:

| File | URL |
|------|-----|
| `src/pages/index.astro` | `/` |
| `src/pages/book-call.astro` | `/book-call/` |
| `src/pages/blog/law-firm-seo.astro` | `/blog/law-firm-seo/` |
| `src/pages/law-firm-seo/benefit.astro` | `/law-firm-seo/benefit/` |

There is no central routes file. If you add `src/pages/foo.astro`, you get `/foo/` automatically after save (dev) or build.

**Trailing slashes:** Internal links in the header/footer often use trailing slashes (`/blog/`). Stay consistent with existing links when adding new ones.

**Examples folder:** Files under `src/pages/examples/` are real routes (e.g. `/examples/homepage-example/`). They exist for template previews and visual QA, not for the public menu. Do not link them from production navigation.

---

## Making common changes

### Change homepage marketing copy

Edit `src/layouts/templates/HomepageTemplate.astro` (or the variant your `index.astro` imports). The live homepage uses `HomepageTemplate` from `src/pages/index.astro`.

There are also `HomepageTemplate-v2.astro` and `HomepageTemplate-v3.astro` plus matching `/examples/homepage-v2-example/` routes, legacy migration experiments. Confirm which template `index.astro` points at before editing.

### Add or edit a blog post

1. Open or create `src/pages/blog/your-slug.astro`.
2. Import `BlogPostTemplate` and set frontmatter props (`seoTitle`, `publishDate`, `chapters`, etc.).
3. Put article HTML between the opening and closing template tags (the default slot).

```astro
---
import BlogPostTemplate from '../../layouts/templates/BlogPostTemplate.astro';
---

<BlogPostTemplate
  seoTitle="Your Title | Constellation Marketing"
  seoDescription="..."
  canonicalUrl="https://goconstellation.com/blog/your-slug/"
  publishDate="2026-05-18"
>
  <h2>First section</h2>
  <p>Body copy...</p>
</BlogPostTemplate>
```

Content is **HTML in the Astro file**, not Markdown. Scraped posts may still contain WordPress artifacts (`&#8217;`, empty `<ol></ol>`, old image classes); clean those as you touch the file.

See `src/pages/examples/blog-post-example.astro` for a reference shape.

### Add or edit a city landing page

City pages use `CityTemplate` with props for city name, state, and image paths:

```astro
---
import CityTemplate from '../layouts/templates/CityTemplate.astro';
---

<CityTemplate
  seoTitle="..."
  city="Austin"
  state="Texas"
  heroImage="/images/attorney-marketing-austin-title.webp"
  ...
/>
```

Images must exist under `public/images/`. For bulk city generation, see `scripts/build_city_pages.py` (optional; paths inside may need updating for your machine).

### Change site-wide header or footer

- `src/components/SiteHeader.astro`: nav links, dropdowns, mobile menu, sticky green pill bar
- `src/components/SiteFooter.astro`: footer columns and legal links

The header includes a small inline `<script>` for dropdown and mobile menu behavior. There is no separate JavaScript bundle; scripts ship as part of the HTML Astro emits.

### Change global SEO shell (title, OG tags, favicon)

Edit `src/layouts/BaseLayout.astro`. Every template that uses `BaseLayout` inherits:

- `<title>`, meta description, canonical
- Open Graph and Twitter tags
- Favicon
- Link to `/css/production-extracted.css`
- Import of `global.css` (Tailwind + Poppins + theme)

Per-page overrides can pass extra markup via `<Fragment slot="head">` or `<style slot="head">` in templates.

---

## Styling while you develop

This project uses **three styling systems at once**. That is intentional for the WordPress migration, not a mistake.

| Layer | Location | Role |
|-------|----------|------|
| **Legacy Divi CSS** | `public/css/production-extracted.css` (~488KB) | Loaded on every page from `BaseLayout`; preserves old Divi class behavior |
| **Tailwind 4** | `src/styles/global.css` | Brand colors, utilities, base typography |
| **Inline `style=""`** | Many templates | Pixel-matched sections copied from production |

`global.css` also imports `divi-theme-options.css` for sitewide custom CSS from WordPress theme options.

**Practical advice:**

- Prefer editing the template’s scoped `<style>` block or Tailwind classes when building **new** UI.
- When matching production WordPress, you may need inline styles or Divi class names; fighting only Tailwind will slow you down.
- After changing `global.css` or files in `public/css/`, hard-refresh the browser (cached CSS is common).

`@fontsource/poppins` is in `package.json`, but headings primarily use **self-hosted** files in `public/fonts/` referenced from `global.css`.

---

## Static assets (`public/`)

Anything in `public/` is served from the site root:

- `/images/...` → `public/images/...`
- `/css/production-extracted.css` → `public/css/production-extracted.css`
- `/fonts/poppins-700.woff2` → `public/fonts/poppins-700.woff2`

Large image trees live under `public/images/` (including `wp-uploads/` from migration). Optimize images before committing huge binaries; city scripts convert to WebP for a reason.

**Do not** import `public/` files from frontmatter as modules; use URL paths as strings in `src=""` or `href=""`.

---

## Third-party embeds (work locally, break in edge cases)

Some pages load external scripts or iframes:

| Page | Integration |
|------|-------------|
| `book-call.astro` | Lead Connector / HighLevel booking iframe + `form_embed.js` |
| `support-center.astro` | Support iframe |
| `PodcastTemplate` | Podcast player iframe |

These usually render in dev, but ad blockers, cookie settings, or domain allowlists can make them look blank locally. If the rest of the page works, test embeds on staging before assuming your markup is wrong.

---

## Optional: visual regression (BackstopJS)

Compares the **staging homepage** to **production** section-by-section (screenshot diffs). Requires `npm run dev` plus `npx backstop test`.

See **[VISUAL-QA.md](./VISUAL-QA.md)** for commands, `data-section` setup, HTML reports, and the optional `fix-loop.mjs` AI workflow.

---

## Optional: Python migration scripts

Located in `scripts/`. They are **offline generators**, not run during `npm run build`.

| Script | Purpose |
|--------|---------|
| `build_blog_posts.py` | Scrape production URLs → write `src/pages/blog/*.astro` |
| `build_astro_post.py` | Convert Firecrawl JSON export → one blog `.astro` file |
| `build_city_pages.py` | Scrape city pages, download/WebP images, write city `.astro` files |
| `split-sections.mjs` | Split a page’s HTML into `sections-output/*.html` for inspection |
| `fix-loop.mjs` | Backstop + Claude homepage iteration |

**Caveats:**

- Several scripts contain **absolute paths** from another developer’s machine (`/Users/patrickcarver/...`). Update paths before running on yours.
- `build_city_pages.py` uses SSL verification bypass for scraping; only use in trusted environments.
- Re-running bulk scrapers can **overwrite** hand-edited pages. Commit or branch first.

For homepage migration from Refore exports, follow [REFORE-APPROACH.md](../REFORE-APPROACH.md).

---

## Environment variables

| Variable | Required? | Used by |
|----------|-----------|---------|
| `ANTHROPIC_API_KEY` | Only for fix-loop | `scripts/fix-loop.mjs` |

Create a `.env` file in the repo root (already in `.gitignore`). Never commit API keys.

Deploy uses **`CLOUDFLARE_API_TOKEN`** in GitHub Actions secrets, not in your local `.env` unless you deploy manually with Wrangler.

---

## Deploy pipeline (what happens after you push)

On push to **`main`**, GitHub Actions builds and deploys to Cloudflare Pages project **`con-staging`**.

See **[DEPLOYMENT.md](./DEPLOYMENT.md)** for the full pipeline, `CLOUDFLARE_API_TOKEN`, manual Wrangler commands, rollback, and production cutover.

---

## Troubleshooting

### `npm run dev` fails immediately

- Confirm Node ≥ 22.12: `node -v`
- Delete `node_modules` and run `npm install` again
- Check for syntax errors in the last `.astro` file you edited (Astro error messages usually name the file and line)

### `npm run build` fails on one page

- Astro fails the **whole** build. Read the stack trace for the file path.
- Common causes: unclosed HTML in a blog post, invalid frontmatter, or a missing import path (`../../layouts/...` depth wrong for nested folders)

### Page looks unstyled or “wrong font”

- Confirm `BaseLayout` is in the chain (templates must use it)
- Check DevTools Network for `production-extracted.css` and `global.css` loading (200, not 404)
- Divi-dependent markup needs Divi CSS classes, stripping classes without replacing styles will look broken

### Images 404

- Path must start with `/images/...` and file must exist under `public/images/`
- Case-sensitive on Linux/CI; macOS dev may hide case mistakes until deploy

### Changes not showing

- Dev server: save the file; hard refresh
- If you only edited `public/`, refresh is enough
- If you edited `astro.config.mjs`, restart `npm run dev`

### Build is slow

- ~400 pages of mostly HTML content is normal
- First build after dependency changes is slower; subsequent builds use Astro cache in `.astro/` (gitignored)

### Header dropdowns don’t work

- Requires the inline script at the bottom of `SiteHeader.astro`
- Check browser console for JS errors on that page

---

## What to avoid

- **Do not** add `output: 'server'` or API routes without a project-wide decision, this repo is static-only.
- **Do not** commit `.env` or Cloudflare tokens.
- **Do not** run bulk Python scrapers on `main` without review; they can overwrite manual fixes.
- **Do not** assume `README.md` at the repo root is accurate, it is still the default Astro starter text; use `documents/` instead.
- **Do not** link `/examples/*` from production navigation.

---

## Suggested workflow for a typical task

1. Pull latest `main`, `npm install` if lockfile changed.
2. `npm run dev` → reproduce the page in the browser.
3. Edit the page or template; refresh until it looks right.
4. `npm run build` → fix any build errors.
5. Optionally `npm run preview` and click through your changes.
6. Commit with a clear message; push to a branch; open PR.
7. After merge to `main`, CI deploys to Cloudflare staging automatically.

For homepage visual parity work, add Backstop reference/test steps from the section above.

---

## Related documentation

| Document | Topics |
|----------|--------|
| [SITE-OVERVIEW.md](./SITE-OVERVIEW.md) | Architecture, templates list, migration summary |
| [ASTRO.md](./ASTRO.md) | Astro concepts, static output, slots, config on this site |
| [REFORE-APPROACH.md](../REFORE-APPROACH.md) | Homepage export from WordPress via Refore |
| `src/pages/examples/` | Live reference pages for each template type |

| [TEMPLATES.md](./TEMPLATES.md) | Every layout template, props, slots, picker guide |
| [CONTENT-GUIDE.md](./CONTENT-GUIDE.md) | Blog posts, city pages, SEO, images |
| [MIGRATION-TOOLING.md](./MIGRATION-TOOLING.md) | Scrapers, Refore, Backstop, fix-loop |

| [STYLING.md](./STYLING.md) | CSS layers, brand palette, template classes, Divi pitfalls |

| [DEPLOYMENT.md](./DEPLOYMENT.md) | Cloudflare Pages, CI/CD, cutover |

| [VISUAL-QA.md](./VISUAL-QA.md) | BackstopJS commands, reports, data-section setup |

---

## Quick reference

```bash
# Daily loop
npm run dev          # http://localhost:4321

# Before PR / after big changes
npm run build
npm run preview      # http://localhost:4321 (production build)

# Visual QA (homepage)
npm run dev          # terminal 1
npx backstop test --config=backstop.config.cjs   # terminal 2
```

**Key files to bookmark:** `astro.config.mjs`, `src/layouts/BaseLayout.astro`, `src/components/SiteHeader.astro`, `src/layouts/templates/HomepageTemplate.astro`, `src/layouts/templates/BlogPostTemplate.astro`, `src/layouts/templates/CityTemplate.astro`.
