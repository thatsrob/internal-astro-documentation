# Astro on This Site

This document explains **what Astro is**, **how this project uses it**, and **why that choice fits** a large marketing site migrating off WordPress.

**New to Astro?** Start with the framework primer in [astro-basics/](./astro-basics/README.md) (language concepts, components, routing, build time). Return here for Constellation-specific configuration and patterns.

For day-to-day commands, see [DEVELOPMENT.md](./DEVELOPMENT.md). For the full project picture, see [COMPREHENSIVE-OVERVIEW.md](./COMPREHENSIVE-OVERVIEW.md).

---

## What is Astro?

[Astro](https://astro.build) is a framework for building **content-focused websites**. Its core idea is simple: ship **as little JavaScript to the browser as possible**, and generate **static HTML** at build time whenever you can.

You write pages in **`.astro` files**. Each file can contain:

1. A **frontmatter** block (`---` at the top), which is JavaScript/TypeScript that runs at build time.
2. A **template**, which is HTML-like markup with expressions and components.
3. Optional **`<style>`** and **`<script>`** scoped to that file.

At build time, Astro runs the frontmatter, renders components to HTML, and writes files to `dist/`. The browser receives normal HTML and CSS, not a heavy client-side app bootstrapping every page.

Official docs: [docs.astro.build](https://docs.astro.build).

**Deeper primer (8 chapters):** [astro-basics/README.md](./astro-basics/README.md) — project layout, props vs slots, build vs runtime, assets, and template directives with more examples than this summary.

---

## Astro basics (concepts you will see everywhere)

### File-based routing

Any file under `src/pages/` becomes a URL:

| File | URL |
|------|-----|
| `src/pages/index.astro` | `/` |
| `src/pages/book-call.astro` | `/book-call/` |
| `src/pages/blog/law-firm-seo.astro` | `/blog/law-firm-seo/` |

Folders create path segments. There is no separate router file; the filesystem is the route table.

### Components

`.astro` files can import other `.astro` files and use them like HTML tags:

```astro
---
import SiteHeader from '../components/SiteHeader.astro';
---
<SiteHeader />
```

Props are passed as attributes. Child markup goes between opening and closing tags (the **default slot**).

### Layouts

A layout is just a component that wraps a page. This site’s root document shell is `BaseLayout.astro`; page-type shells live in `src/layouts/templates/`.

### Frontmatter runs at build time

Code in the `---` block runs **once per page at build time**, not on every visitor request:

```astro
---
const formattedDate = publishDate
  ? new Date(publishDate).toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' })
  : null;
---
```

That is how templates format dates, merge props, and prepare data without a server.

### Slots

Layouts expose placeholders for child content:

```astro
<!-- BaseLayout.astro -->
<body>
  <slot />
</body>
```

```astro
<!-- Named slot for extra <head> content -->
<slot name="head" />
```

A page or template passes content into slots:

```astro
<BaseLayout title="...">
  <style slot="head">body { background: #efefef; }</style>
  <main>...</main>
</BaseLayout>
```

`BlogPostTemplate` uses a default slot for the article body: everything between `<BlogPostTemplate>...</BlogPostTemplate>` in the page file becomes the article HTML.

### `Astro` global

Inside frontmatter and templates you get helpers such as:

| API | Typical use on this site |
|-----|---------------------------|
| `Astro.props` | Props passed from parent component |
| `Astro.url` | Current page URL (dev/build); used as default canonical fallback in `BaseLayout` |
| `Astro.site` | Value of `site` from `astro.config.mjs` (origin for absolute URLs) |
| `Astro.params` | Dynamic route segments from `[slug]` filenames |
| `Astro.slots.has('name')` | Whether parent passed a named slot |

### Static output

This project sets:

```js
// astro.config.mjs
output: 'static',
```

That means **every route is pre-rendered to HTML at build time**. There are no Astro server endpoints, no SSR per request, and no `getServerSideProps`-style runtime. The output is a folder of files Cloudflare Pages can serve from a CDN.

### What this project does *not* use (yet)

These are valid Astro features but **not part of this codebase today**:

| Feature | Status here |
|---------|-------------|
| React / Vue / Svelte islands (`client:load`, etc.) | Not used (zero client directives) |
| Content Collections (`src/content/`) | Not used; blog HTML lives in page files |
| MDX | Not used |
| Server islands / SSR (`output: 'server'`) | Not used |
| API routes (`src/pages/api/`) | Not used |

The site is intentionally **HTML-first**, which matches a migrated marketing site with minimal interactivity beyond third-party embeds (booking calendar, ClickUp form, YouTube).

### Content model here (not Content Collections)

Many Astro tutorials store posts in `src/content/` as Markdown with typed schemas. **We do not.** Each blog or city URL is its own file under `src/pages/`, with HTML in the template slot. That matches a WordPress migration (one URL → one file) and keeps grep and PR diffs straightforward, at the cost of larger files.

---

## How Astro is configured here

**Config file:** `astro.config.mjs`

```js
export default defineConfig({
  output: 'static',
  site: 'https://www.goconstellation.com',

  redirects: { /* legacy WordPress slugs → shorter paths */ },

  vite: {
    plugins: [tailwindcss()],
  },
});
```

| Setting | Purpose |
|---------|---------|
| `output: 'static'` | Pre-render all ~400 pages to `dist/` |
| `site` | Base URL for Astro features that need an origin (e.g. future sitemap integration) |
| `redirects` | Build-time redirects for renamed URLs (see [SEO.md](./SEO.md)) |
| `vite.plugins` | Tailwind CSS 4 via `@tailwindcss/vite` |

**`tsconfig.json`** extends Astro’s strict preset and includes `src/`. Path aliases are optional; most imports use relative paths like `../../layouts/templates/BlogPostTemplate.astro`.

**Dependencies** (`package.json`): Astro 6, Tailwind 4. No UI framework packages. `@anthropic-ai/sdk` is for a dev script only, not the public site.

**Commands:**

| Command | What happens |
|---------|----------------|
| `npm run dev` | Astro dev server with hot reload (`localhost:4321`) |
| `npm run build` | Writes static site to `dist/` |
| `npm run preview` | Serves `dist/` locally (production-like) |

CI runs only `npm run build`, then deploys `dist/` to Cloudflare Pages.

---

## Architecture on this site

### The page → template → layout stack

Almost every route follows this pattern:

```
src/pages/some-page.astro     ← route + SEO props + content
        ↓
src/layouts/templates/...     ← page-type chrome (hero, article shell, city sections)
        ↓
BaseLayout.astro              ← <html>, <head>, meta, global CSS links
SiteHeader.astro              ← navigation
[content / slot]
SiteFooter.astro              ← footer
```

**Thin page file** (`src/pages/index.astro`):

```astro
---
import HomepageTemplate from '../layouts/templates/HomepageTemplate.astro';
---
<HomepageTemplate
  seoTitle="..."
  seoDescription="..."
  canonicalUrl="https://goconstellation.com/"
/>
```

**Blog post page:** same idea, plus HTML body inside the template:

```astro
---
import BlogPostTemplate from '../../layouts/templates/BlogPostTemplate.astro';
---
<BlogPostTemplate
  seoTitle="Winning Law Firm SEO: The Definitive Guide"
  canonicalUrl="https://goconstellation.com/law-firm-seo/"
  chapters={[...]}
>
  <h2>...</h2>
  <p>...</p>
</BlogPostTemplate>
```

The template maps `seoTitle` → `title` on `BaseLayout` and renders the slot inside `<article class="article-body">`.

### `BaseLayout`: the HTML document

`src/layouts/BaseLayout.astro` is the single place for:

- `<title>`, meta description, canonical link  
- Open Graph and Twitter Card tags  
- Favicon and font preload  
- Import of `src/styles/global.css` (Tailwind + tokens)  
- Link to legacy `public/css/production-extracted.css`  
- Named `head` slot for per-page `<style>` overrides  

```astro
---
import '../styles/global.css';
const { title, description = '', canonicalUrl = Astro.url.href, ... } = Astro.props;
---
<!DOCTYPE html>
<html lang="en-US">
<head>
  <title>{title}</title>
  ...
  <slot name="head" />
</head>
<body>
  <slot />
</body>
</html>
```

### Shared UI components

Only two shared components exist:

- `src/components/SiteHeader.astro`: main navigation  
- `src/components/SiteFooter.astro`: footer links and social icons  

Everything else is template-specific markup (often migrated from WordPress/Divi).

### Styling with Astro

Three mechanisms work together:

1. **Imported global CSS:** `global.css` in `BaseLayout` (Tailwind, fonts, theme).  
2. **Static public CSS:** `production-extracted.css` linked in `<head>`.  
3. **Scoped / slotted styles:** `<style>` in templates, often with `slot="head"` for page background overrides.

Example from `BlogPostTemplate`:

```astro
<style is:global slot="head">
  body { background: #efefef; }
  .article-body h2 { ... }
</style>
```

`is:global` means those selectors apply outside the component’s default scope, which is needed when styling HTML coming from the default slot.

See [STYLING.md](./STYLING.md) for the full CSS story.

### Third-party scripts and embeds

Astro does not block iframes or external scripts in templates. This site uses them sparingly on specific pages, such as LeadConnector on `book-call.astro` and ClickUp on `support-center.astro`. Those load in the browser like any static HTML page; Astro does not bundle them.

Sitewide analytics (GTM) is **not** in `BaseLayout` yet; production WordPress still owns that until launch. See [INTEGRATIONS.md](./INTEGRATIONS.md).

### `public/` vs `src/`

| Directory | Behavior |
|-----------|----------|
| `src/` | Processed by Astro/Vite (components, bundled CSS imports) |
| `public/` | Copied verbatim to site root (`/images/...`, `/css/...`, `/fonts/...`, `robots.txt`) |

Images and legacy CSS are referenced with root-relative paths: `/images/hero.webp`.

Framework primer: [astro-basics/08-assets-images-and-styles.md](./astro-basics/08-assets-images-and-styles.md).

---

## Benefits of Astro for this project

### 1. Performance by default

Marketing sites win when pages load fast. Astro ships **static HTML** with **no framework runtime** on most pages. Visitors get content immediately; you are not paying a JavaScript tax for a site that is mostly articles, city landing pages, and lead-gen copy.

That matters for SEO (Core Web Vitals), mobile attorneys browsing on slow connections, and Cloudflare edge caching.

### 2. Predictable builds at scale

~400 routes all compile to files in `dist/`. The build either passes or fails in CI, so there are no runtime surprises from a CMS plugin or PHP version. `npm run build` on `main` is the gate before deploy.

### 3. Git as the source of truth

Content and layout live in the repo. Changes are reviewable in PRs, diffable, and revertible. For a migration from WordPress, that replaces opaque database + plugin state with something the dev team can reason about.

### 4. Simple hosting

`output: 'static'` maps directly to **Cloudflare Pages**: upload `dist/`, no Node server to maintain, no WordPress security patches on the public edge. Staging deploys on every push to `main` via GitHub Actions + Wrangler.

### 5. Gradual migration fit

WordPress exports became `.astro` files with HTML bodies and Divi CSS, not a big-bang rewrite into React. Astro tolerates **lots of static HTML** inside components without forcing a new component model. Templates (`CityTemplate`, `BlogPostTemplate`) give structure where repetition helps; one-off pages (`book-call.astro`) can stay hand-built.

### 6. TypeScript-ready frontmatter

Props interfaces on templates (`interface Props { seoTitle: string; ... }`) document what each page type needs and catch mistakes in the editor. Types add no runtime cost; they strip away at build.

### 7. Low JavaScript surface area

No `client:*` directives means no hydration mismatches, no island architecture to debug, and no second rendering model. Interactivity that exists (booking widget, YouTube) is isolated to vendor embeds.

---

## Astro vs what we left behind (WordPress)

| Aspect | WordPress + Divi | Astro (this repo) |
|--------|------------------|---------------------|
| Runtime | PHP + DB per request | Pre-built HTML on CDN |
| Content | DB + admin UI | `.astro` files in Git |
| Plugins | Forms, SEO, caching plugins | Explicit embeds + meta in layouts |
| Styling | Divi builder + theme options | Templates + legacy CSS + Tailwind |
| Deploy | Hosted WP stack | `dist/` to Cloudflare Pages |
| Scale cost | DB + PHP under traffic spikes | Static file serving |

Tradeoff: non-developers cannot publish from wp-admin without a workflow change (edit repo or a future CMS). The team accepted that for performance, security, and version control.

---

## Mental model for debugging

When something is wrong, classify it:

| Symptom | Likely layer |
|---------|----------------|
| 404 on a path | Missing or misnamed file under `src/pages/` |
| Wrong title / meta | Page props → template → `BaseLayout` prop names |
| Layout broken | Template markup or Divi CSS specificity |
| Style not applying | Wrong CSS layer (Divi vs `global.css` vs inline) |
| Works in `dev` but not `preview` | Run `build`; production bundling differs |
| Embed blank on staging | Third-party domain allowlist, not Astro |
| Build fails | Syntax in `.astro` or invalid HTML in template |

**`npm run dev`** compiles on demand. **`npm run build`** is what CI and Cloudflare use. Always verify with build + preview before merging large changes.

---

## Common Astro patterns in this repo

### Passing SEO from page to layout

Page sets `seoTitle`; template passes `title={seoTitle}` to `BaseLayout`. Always match the prop name `BaseLayout` expects (`title`, not `seoTitle`). See the known `CoreTemplate` issue in [COMPREHENSIVE-OVERVIEW.md](./COMPREHENSIVE-OVERVIEW.md).

### Lists and `.map()` in templates

City cards, nav items, and testimonials use Astro expressions in markup:

```astro
{trustSignals.map(s => (
  <div>{s}</div>
))}
```

In this repo, map to normal HTML elements, e.g. `{items.map(item => <li>{item}</li>)}`.

### Conditional rendering

```astro
{formattedDate && (
  <span>{formattedDate}</span>
)}
```

### `set:html` for SVG strings

Some pages inject icon markup from string variables with `set:html={p.icon}`. Use this carefully; only trusted strings.

### Example routes

`src/pages/examples/*.astro` are real builds used to preview templates (`/examples/blog-post-example/`). They are for dev/QA, not production nav. See [TEMPLATES.md](./TEMPLATES.md).

---

## What to read next

| Topic | Document |
|-------|----------|
| Astro fundamentals (8-part primer) | [astro-basics/](./astro-basics/README.md) |
| Commands and editing workflow | [DEVELOPMENT.md](./DEVELOPMENT.md) |
| Template props and picker | [TEMPLATES.md](./TEMPLATES.md) |
| Canonicals and redirects | [SEO.md](./SEO.md) |
| CSS layers | [STYLING.md](./STYLING.md) |
| Embeds and GTM gap | [INTEGRATIONS.md](./INTEGRATIONS.md) |
| Build and deploy | [DEPLOYMENT.md](./DEPLOYMENT.md) |
| New developer onboarding | [COMPREHENSIVE-OVERVIEW.md](./COMPREHENSIVE-OVERVIEW.md) |

---

## Further learning (official Astro)

- [Getting started](https://docs.astro.build/en/getting-started/)
- [Pages and routing](https://docs.astro.build/en/basics/astro-pages/)
- [Components](https://docs.astro.build/en/basics/astro-components/)
- [Layouts](https://docs.astro.build/en/basics/layouts/)
- [Static site generation](https://docs.astro.build/en/guides/static-sites/)
- [Configuration reference](https://docs.astro.build/en/reference/configuration-reference/)

Those cover Astro in general; the sections above describe **how this repository applies** those ideas to Constellation Marketing’s site rebuild.
