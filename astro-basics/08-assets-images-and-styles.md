# 8. Assets, images, and styles

How Astro handles **static files**, **images**, and **CSS**—and how that maps to a typical marketing site.

**Back to series:** [Astro basics index](./README.md) · **Previous:** [Template expressions](./07-template-expressions.md)

---

## Two places for files

| Location | Processed by Vite? | URL in browser | Typical use |
|----------|-------------------|----------------|-------------|
| `public/` | No — copied as-is | `/images/logo.svg` | Images, fonts, legacy CSS, `robots.txt` |
| `src/` | Yes — imports, hashing, bundling | Depends on import | Component CSS, assets referenced from `.astro` |

**Rule of thumb:** If you need a **stable path** (SEO images, `/fonts/`, third-party docs linking to `/images/foo.webp`), put it in **`public/`**. If the file is only used inside components and can be hashed, import from **`src/`**.

Constellation’s migrated site leans heavily on **`public/images/`** and **`public/css/`** so paths match what WordPress used.

---

## `public/` — static assets

Anything in `public/` is served from the site root:

| File on disk | URL |
|--------------|-----|
| `public/images/hero.webp` | `/images/hero.webp` |
| `public/fonts/poppins.woff2` | `/fonts/poppins.woff2` |
| `public/robots.txt` | `/robots.txt` |

Reference in HTML or CSS with a **root-relative** path (leading slash):

```astro
<img src="/images/attorney-marketing-austin-title.webp" alt="Austin law firm marketing" />
```

```css
.hero {
  background-image: url('/images/hero.webp');
}
```

Paths are **not** rewritten at build time. What you write is what visitors request.

### Image attributes worth setting

```astro
<img
  src="/images/hero.webp"
  alt="Law firm marketing team"
  width="1200"
  height="630"
  loading="lazy"
  decoding="async"
/>
```

| Attribute | Why |
|-----------|-----|
| `alt` | Accessibility and SEO; required unless decorative |
| `width` / `height` | Reduces layout shift (CLS) |
| `loading="lazy"` | Defer off-screen images |
| `fetchpriority="high"` | LCP hero only (sparingly) |

Open Graph images (`og:image`) are usually absolute URLs in `BaseLayout` props, often pointing at `/images/constellation-og.jpg` or a page-specific asset in `public/`.

---

## Importing from `src/`

You can import files from `src/` in frontmatter. Vite processes them and may add a content hash to the filename in `dist/`:

```astro
---
import logo from '../assets/logo.svg';
---
<img src={logo.src} alt="Constellation" width={logo.width} height={logo.height} />
```

| Import type | What you get |
|-------------|--------------|
| Images | Object with `.src`, `.width`, `.height` (and format metadata) |
| CSS | Styles injected when the component is used |
| Raw text | String contents |

Use this when you want build-time optimization and don’t need a fixed URL. Our repo uses it sparingly (e.g. `global.css` imported in `BaseLayout`).

---

## The `<Image />` component (optional)

Astro ships an **`@astrojs/image`** integration (or built-in `astro:assets` in newer versions) for responsive images, WebP conversion, and `width`/`height` to reduce layout shift.

```astro
---
import { Image } from 'astro:assets';
import hero from '../assets/hero.jpg';
---
<Image src={hero} alt="Hero" widths={[400, 800, 1200]} />
```

**Constellation today:** most pages use plain `<img src="/images/...">` in `public/`, not the Image component. That is fine for a static marketing site; migration scripts already output WebP paths where needed.

See [IMAGE-MIGRATION.md](../IMAGE-MIGRATION.md) for moving assets off WordPress.

---

## Images in markdown and HTML bodies

Blog and city pages often contain **inline HTML** inside a template slot:

```astro
<BlogPostTemplate ...>
  <p><img src="/images/blog/diagram.webp" alt="..." /></p>
</BlogPostTemplate>
```

Those `src` values are plain strings—Astro does not rewrite them. You are responsible for:

1. Putting the file in `public/images/...`
2. Using the correct path in the markup
3. Adding `alt` text (accessibility)

To find broken or remote images:

```bash
grep -r 'src="http' src/pages/ --include='*.astro'
grep -r 'wp-content/uploads' src/ --include='*.astro'
```

---

## CSS in Astro (three patterns)

### 1. Global stylesheet import (frontmatter)

```astro
---
import '../styles/global.css';
---
```

Bundled once; rules apply site-wide. Our `BaseLayout` imports Tailwind entry CSS this way.

### 2. Scoped `<style>` in a component

```astro
<style>
  .card { padding: 1rem; }
</style>
```

Astro adds a unique attribute so `.card` only affects markup in **this** component’s template. Good for isolated widgets.

### 3. Global styles in a component (`is:global`)

```astro
<style is:global>
  .article-body h2 { margin-top: 2rem; }
</style>
```

Needed when styling **slot content** (HTML passed from the parent page). Scoped styles do not reach children injected through `<slot />`.

**Named head slot** (used on our blog template):

```astro
<style is:global slot="head">
  body { background: #efefef; }
</style>
```

Injects into `<head>` via `<slot name="head" />` on the layout.

---

## CSS variables from frontmatter

Pass build-time values into CSS:

```astro
---
const accent = '#1a3c34';
---
<div class="banner">...</div>

<style define:vars={{ accent }}>
  .banner { border-color: var(--accent); }
</style>
```

Variables are resolved at build time, not in the browser from live data.

---

## Legacy CSS alongside Astro

Real migrations often keep a **large legacy stylesheet** in `public/css/` and link it from the layout:

```astro
<link rel="stylesheet" href="/css/production-extracted.css" />
```

That coexists with Tailwind and component styles. Specificity and load order matter—see [STYLING.md](../STYLING.md) and [PERFORMANCE-AND-CSS.md](../PERFORMANCE-AND-CSS.md).

---

## Fonts

| Approach | Example |
|----------|---------|
| Self-hosted in `public/fonts/` | `@font-face` in CSS → `/fonts/poppins.woff2` |
| Google Fonts link in layout | `<link href="https://fonts.googleapis.com/..." />` |
| `fontsource` npm package | Import in frontmatter (bundled) |

Constellation uses self-hosted Poppins under `public/fonts/` with preload in `BaseLayout`.

```astro
<link rel="preload" href="/fonts/poppins-600.woff2" as="font" type="font/woff2" crossorigin />
```

Preload only fonts actually used above the fold—each preload competes for bandwidth.

### Tailwind inside Astro

This repo loads Tailwind from `src/styles/global.css` imported in `BaseLayout`. Use utility classes in templates and slot HTML:

```astro
<section class="mx-auto max-w-4xl px-4 py-12">
```

Tailwind scans source files at build time; new classes in a page file are picked up on next `dev`/`build`. Legacy Divi CSS may override utilities where specificity is higher—see [STYLING.md](../STYLING.md).

---

## Third-party assets

Scripts, iframes, and widgets (YouTube, booking calendars, forms) are **normal HTML** in templates. Astro does not bundle them unless you import them.

```astro
<iframe src="https://www.youtube.com/embed/..." title="Episode"></iframe>
```

Failures are usually **domain allowlists** or CSP on the host—not Astro build errors.

---

## Build output: what lands in `dist/`

After `npm run build`:

| Source | In `dist/` |
|--------|------------|
| `public/**` | Copied unchanged to root |
| Imported `src` assets | Hashed filenames under `_astro/` (or similar) |
| Compiled pages | `index.html` per route |
| Global + scoped CSS | Bundled CSS files or inlined |

Preview locally:

```bash
npm run build && npm run preview
```

Use this when an image or style works in `dev` but not after build.

---

## Checklist for a new image

1. Save file under `public/images/...` (or run migration script).
2. Reference as `/images/...` in `.astro` or slot HTML.
3. Set meaningful `alt` (empty `alt=""` only for decorative images).
4. Prefer WebP (or AVIF) for photos; SVG for logos/icons.
5. Run `npm run build` and open the page in `preview`.
6. Confirm the file exists in `dist/images/...`.

---

## What we skip on this site (for now)

| Feature | Status |
|---------|--------|
| `astro:assets` `<Image />` everywhere | Optional; mostly plain `<img>` |
| Content Collections for images | Not used |
| CMS media library | Content in Git + `public/` |

### CSS load order on our site (why styles “lose”)

Rough order in `<head>`:

1. `global.css` (Tailwind + tokens) — imported via Astro
2. `production-extracted.css` — linked from `public/`
3. Per-page `<style slot="head">` overrides

Later rules and more specific selectors win. If a Divi rule beats a Tailwind utility, fix in the right layer or increase specificity deliberately—not random `!important` chains.

---

## Series complete

You now have the vocabulary to read any `.astro` file in the Constellation repo.

| Next step | Document |
|-----------|----------|
| How Astro applies to our site | [ASTRO.md](../ASTRO.md) |
| WordPress image cleanup | [IMAGE-MIGRATION.md](../IMAGE-MIGRATION.md) |
| CSS layers on this site | [STYLING.md](../STYLING.md) |
| Run and edit locally | [DEVELOPMENT.md](../DEVELOPMENT.md) |
| Pick a page template | [TEMPLATES.md](../TEMPLATES.md) |
